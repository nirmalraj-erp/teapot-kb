# eu-central-1

Second production host — a separate physical Lightsail box from `ap-southeast-1`.
IP `3.125.249.119`, instance `i-0eaccc33083c434aa`.

SSH connection details: `tpt-dev-scripts`' gitignored `.env` (`TPT_EU_SERVER_HOST` /
`TPT_EU_SERVER_USER` / `TPT_EU_SERVER_SSH_KEY`) — never copied here.

`tpt-dev-scripts` is cloned here too, at the same `/home/ubuntu/tpt/17.0/tpt-dev-scripts/` path.
Most `odoo-conf/`/`systemd/` files are shared verbatim with `ap-southeast-1` — **port numbers
are only unique per host, not globally across both**.

## Clients (as of last verification 2026-09-02)

| Client | Domain(s) | Status |
|---|---|---|
| `ergodenta` | `ergodenta.com`, `ergodenta.dk` | Running (prod) |
| `madental` | `madental.dk` | Running (prod) |
| `dpc-dental` | — | Running (**live here**; the `ap-southeast-1` deployment of the same client is dormant) |

`ergodenta` and `madental` addons are developed out of the `odoo-17/tpt-eu` workspace (own
`CLAUDE.md`) — this file only covers their *deploy* target, not their module conventions.

`dpc-dental` is split per host, not shared: this host uses `odoo-conf/dpc-dental-eu-17c.conf` /
`systemd/odoo17-dpc-dental-eu.service` (its own port pair) — do not confuse with the
`ap-southeast-1` pair (`dpc-dental-17c.conf` / `odoo17-dpc-dental.service`), and don't re-merge
them; they were deliberately split 2026-09-02 after the two hosts' shared file drifted and nearly
flipped this host's live port on restart.

A staging instance (`ergodenta-test`, `test-odoo.ergodenta.dk`) existed through 2026-09-02 and
was then fully retired (service, nginx vhost, TLS cert removed). Its `17ergodenta_test_demo`
database was deliberately left in place — don't recreate the staging setup without checking why
it was retired first.

**Authoritative source for ports and current status:** `tpt-dev-scripts/odoo-conf-summary.md`
("EU host" section).

## Known quirks

- Two independent certbot renewal timers were found live on this host at once (apt-packaged
  `certbot.timer` and a separately-installed `snap.certbot.renew.timer`) — see
  `runbooks/certbot-duplicate-timers.md` before touching TLS here.
- A legacy systemd unit (`ergodenta.service`/`madental.service`, since retired) wasn't
  discoverable by searching unit names for "odoo" — same caution as `ap-southeast-1`: verify via
  `/proc/<pid>/cgroup` or unit file contents, not naming assumptions.
