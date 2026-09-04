---
name: tpt-odoo-ops
description: Runbook for deploying and operating Teapot Techies' production Odoo 17 instances (ergodenta, madental, and other clients) across the ap-southeast-1 and eu-central-1 hosts — deploying addon code, cutting a client over to systemd, retiring a client, log rotation, certbot hygiene, onboarding a new client. Use whenever asked to deploy, restart, migrate, retire, or troubleshoot a TPT Odoo production instance, or to add/change server-side deploy config.
---

# TPT Odoo deploy operations

## Where things live

- Addon code for **ergodenta**/**madental** is this (`tpt-eu`) working directory's `ergodenta/`
  and `madental/` repos. Other clients' addon repos exist only on the servers — no local
  checkout.
- Deploy config — `odoo-conf/*.conf`, `nginx-conf/`, `systemd/*.service`, `logrotate/` — lives
  in the separate `~/PycharmProjects/tpt-dev-scripts` repo. **Read its `CLAUDE.md` first** for
  full conventions (port tiers, memory limits, branch/commit style); this skill is the
  procedure layer on top of those facts.
- Two independent production hosts, `ap-southeast-1` and `eu-central-1`. SSH host/user/key for
  both are in `tpt-dev-scripts/.env` (gitignored) as `TPT_SERVER_*` / `TPT_EU_SERVER_*`. Port
  numbers are only unique **within** a host, never assume global uniqueness.

## Before touching any running process — check for hidden supervision first

**Never `kill` a process on these hosts without first ruling out systemd ownership.** A unit
does not have to have "odoo" anywhere in its name or description to be tracking an Odoo
process — grepping `systemctl list-units`/unit filenames for "odoo" is not reliable. Before
killing anything that looks like a bare/unsupervised process:

```bash
cat /proc/<pid>/cgroup          # look for a *.service (not just *.scope) line
systemctl status <pid>          # unambiguous either way
```

If it's genuinely under systemd, use `systemctl stop <unit>` instead of `kill` — a raw `kill`
on a systemd-tracked process with `Restart=always`/`on-failure` triggers an automatic respawn
that races whatever you do next for the port. This has happened twice on this project
(`ergodenta.service`/`madental.service` on the EU host, discovered mid-migration) — it's cheap
to check and expensive to un-race.

## Deploying an addon code change to a client

1. Get the right host's SSH details from `tpt-dev-scripts/.env`.
2. SSH in, `cd` to the client's addon repo on the server (e.g.
   `/home/ubuntu/tpt/17.0/madental`), and run `git status --short` **before** pulling — a dirty
   tree means someone made an uncommitted server-side edit; investigate rather than overwrite it.
3. `git pull`. If it fails with `Permission denied (publickey)`, the default SSH identity isn't
   registered for that repo's remote — retry with:
   ```bash
   GIT_SSH_COMMAND='ssh -i ~/.ssh/bitbuket -o StrictHostKeyChecking=accept-new' git pull
   ```
   (this deploy key has worked on both hosts so far).
4. `sudo systemctl restart odoo17-<name>`, then verify: `systemctl is-active odoo17-<name>`,
   `curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:<port>/web/login`, and skim
   `journalctl -u odoo17-<name> -n 30 --no-pager` for tracebacks.
5. A brand-new module in the pulled diff is not *installed* just because the service restarted
   — say so explicitly if the deploy includes one; making it live needs `-i <module>` or the
   Apps UI, a separate step from deploying the code.

## Cutting a bare/legacy process over to a proper systemd unit

Odoo binds one port per instance, so old and new can't run side-by-side to verify a clean
handoff — a genuine few-seconds gap of downtime is expected, not a bug to route around.

1. Confirm the unit file matches the established pattern (see any `tpt-dev-scripts/systemd/
   odoo17-*.service` as a template): absolute `venv17` interpreter path, `WorkingDirectory=
   .../odoo`, `ExecStart -c` pointing at the *repo's* `odoo-conf/<name>.conf` (not a server-only
   ad hoc path), `Restart=on-failure` + `RestartSec=10` + `StartLimitBurst=5`/
   `StartLimitIntervalSec=300`.
2. `sudo cp` it into `/etc/systemd/system/`, `sudo systemctl daemon-reload`.
3. Stop the old process — `systemctl stop <old-unit>` if you found one in the check above,
   otherwise `sudo kill -TERM <pid>` and confirm exit (`kill -0 <pid>` loop) and the port is
   free (`sudo ss -tlnp | grep :<port>`).
4. `sudo systemctl enable --now odoo17-<name>`, then verify: port bound to the *new* PID, `curl`
   returns 200, journal is clean.
5. **If a second process grabs the port the instant you free it, stop and investigate before
   doing anything else** — something else is racing you for it (almost certainly a hidden unit
   you haven't found yet; go back to the check above rather than assuming it'll sort itself out).

## Retiring a client

Confirm scope with the user before starting — "retire" can mean anything from "just stop the
service" to "delete everything including the database," and dropping data is never something
to infer from a casual instruction.

1. `sudo systemctl stop`/`disable` the unit; remove the deployed unit file from
   `/etc/systemd/system/` (or leave it disabled in place if minimal scope was requested).
2. Remove the nginx vhost from `/etc/nginx/sites-enabled/`, `sudo nginx -t`, `sudo systemctl
   reload nginx`.
3. `sudo certbot delete --cert-name <domain> --non-interactive` — check which certbot binary is
   actually authoritative on that host first (see the certbot section below); using the wrong
   one may not see the cert at all.
4. `git rm` the corresponding `odoo-conf/`, `systemd/`, `nginx-conf/sites-enabled/` entries from
   `tpt-dev-scripts`, update `odoo-conf-summary.md`'s table/notes, commit, push, then `git pull`
   on every host that clones this repo (there are at least two).
5. Leave the database alone unless dropping it was explicitly part of the agreed scope.

## Setting up log rotation for a client that doesn't have it

- Use `copytruncate` — Odoo doesn't reopen its log file handle on a signal, so a rename-based
  rotation would leave it writing into the old, now-unlinked file forever.
- **Don't add `delaycompress`** alongside `copytruncate`. `delaycompress` exists to protect a
  renamed file that a process might still be writing to before it reopens its handle — that
  scenario doesn't exist with `copytruncate` (the live file is truncated in place, never
  renamed). Adding it anyway just leaves a freshly-rotated multi-GB log sitting uncompressed
  for a full extra rotation cycle for no benefit.
- If the log's directory is owned by a non-root user and group-writable, logrotate refuses to
  touch it by default ("insecure permissions") unless the stanza has `su <owner-user>
  <owner-group>`.
- A forced first rotation (`sudo logrotate -f /etc/logrotate.d/<name>`) on a multi-GB log is
  genuinely I/O-bound and can take well past a couple of minutes on these boxes — that's not a
  stalled command. Check `ps aux | grep logrotate` before assuming it failed and retrying (a
  retry while the first one is still running will find the log already mid-rotation).

## Certbot: check for duplicates before touching anything

`systemctl list-timers | grep certbot` — if more than one timer fires, **don't assume the
client with the higher version number is automatically the safe one to keep**. One host had
both an apt-packaged `certbot.timer` (in continuous use for a year) and a `snap.certbot.
renew.timer` (installed independently, much later) both live — and the *older* apt client
turned out to be the one with the actual renewal history for at least one cert. Before masking
either:

1. Check each `/etc/letsencrypt/renewal/<domain>.conf`'s `version =` line against which
   client's version that corresponds to, to see which one last actually renewed that
   *specific* cert (not just "checked and found not due").
2. Confirm the client you intend to *keep* can successfully renew **every** cert on that host:
   ```bash
   sudo <certbot-binary-or-'snap run certbot'> renew --dry-run --no-random-sleep-on-renew
   ```
   Without `--no-random-sleep-on-renew`, `renew` injects a genuine multi-minute random delay
   per certificate before doing anything (deliberate load-spreading against Let's Encrypt,
   not a bug) — that's normal, but it makes an interactive test look hung for several minutes
   if you don't pass the flag.
3. Only then `systemctl disable --now` and `systemctl mask` the redundant one — mask, so a
   future package upgrade's postinst script can't silently re-enable it.

## Adding a new client

Copy an existing `odoo-conf/*.conf` and `systemd/odoo17-*.service` from a client with a similar
workload (standard backend / website-only / POS / purchase-heavy — see the memory-tier guidance
in `tpt-dev-scripts/CLAUDE.md`) rather than starting from the generic `odoo-run-tmpl.conf`/
`odoo_template.service`. Check the target host's port table in `odoo-conf-summary.md` before
assigning `xmlrpc_port`/`gevent_port` — collisions are per-host, and a shared config file used
across two hosts (like `dpc-dental` was, before being split 2026-09-02) is a trap: a `git pull`
on one host can silently bring in a port reassignment made for the *other* host's copy of the
same file.
