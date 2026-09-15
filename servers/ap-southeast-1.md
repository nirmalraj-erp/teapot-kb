# ap-southeast-1

Primary production Lightsail host. 7.7 GiB RAM, **2 vCPUs**, 4 GiB swap (added as a backstop).
CPU, not memory, is the binding constraint — see the memory/worker allocation plan in
`tpt-dev-scripts/odoo-conf-summary.md` for why only 3 clients run in worker mode.

SSH connection details: `tpt-dev-scripts`' gitignored `.env` (`TPT_SERVER_HOST` /
`TPT_SERVER_USER` / `TPT_SERVER_SSH_KEY`) — never copied here.

`tpt-dev-scripts` is cloned live on this host at `/home/ubuntu/tpt/17.0/tpt-dev-scripts/`; its
`odoo-conf/*.conf` is what every running instance actually reads. There's also a stale, unrelated
`odoo-conf/` at `/home/ubuntu/tpt/17.0/odoo-conf/` (different repo, dirty tree, nothing reads it)
— don't confuse the two on this box.

## Clients (as of last verification 2026-09-15)

| Client | Workload | Status |
|---|---|---|
| `skenmar` + `kships` | Website only | Running — **merged 2026-09-15** into one process, `odoo17-kships-skenmar.service` (see below) |
| `nvt` | POS | Running (worker mode) |
| `mivik` | Standard backend | Running |
| `blaze-xpert-academy` | Standard backend | Running |
| `gadgets` | POS | Running (worker mode) |
| `samaran` | Standard backend | Running |
| `chakra` | Purchase-heavy backend | Running (worker mode) |
| `saizen` | Website only | **Fully retired 2026-09-15** — conf removed, databases dropped (was already inactive since 2026-08-17) |
| `sanyo` | Website only | Dormant (configured, not started) |
| `dpc-dental` | Standard backend | Dormant on this host (live deployment is on `eu-central-1` instead — see there) |

**`kships`+`skenmar` merge (2026-09-15):** both were threaded (`workers=0`) website-only clients,
so they now share one process (`odoo17-kships-skenmar.service`, `kships-skenmar-17c.conf`) instead
of two — `dbfilter` picks the right database per request instead of a pinned `db_name`. Each
domain's nginx vhost rewrites *both* `Host` and `X-Forwarded-Host` to a synthetic per-site value
(`proxy_mode=True` makes Odoo prefer the latter for this, so rewriting `Host` alone silently does
nothing). `nvt`+`gadgets` were deliberately **not** merged the same way — both are already
`workers=1` POS terminals, and sharing one worker would have reduced real concurrency for exactly
the two clients that prompted this investigation; see `odoo-conf-summary.md` for the full
reasoning.

Also retired 2026-09-15, none of these ever had a running client instance on this host (leftover
Postgres databases only, cleaned up as part of the security incident below):
`raybright`, `viable`, `17focus`, `aversan-em-demo`, `viswox17`.

Only `skenmar`, `kships`, `nvt`, `mivik`, `blaze-xpert-academy`, and `saizen` (inactive) have a
real domain + nginx vhost; `gadgets`, `samaran`, `chakra` are IP/port-only.

**Authoritative source for ports, memory limits, and current status:**
`tpt-dev-scripts/odoo-conf-summary.md` — check there before touching anything, this file will
drift otherwise. Per-client engagement/business context lives in `clients/`.

## Known quirks

- Check for duplicate certbot renewal timers before touching TLS on this host (see
  `runbooks/certbot-duplicate-timers.md`) — confirmed present on the EU host, worth checking here
  too rather than assuming it's clean.
- AWS Lightsail's built-in instance firewall is **inbound-only** — there is no way to block
  outbound/egress traffic from there. Egress control requires enabling VPC peering for the
  account and adding rules to the Network ACL of the peered VPC's subnet instead. Learned the hard
  way during the 2026-09-15 incident below.
- A high-CPU `needrestart` process on this host is not automatically suspicious — it's a real,
  legitimate tool that can hang for an extended period as a side effect of `unattended-upgrade`
  processing a large patch backlog (this host hadn't rebooted in 615+ days as of 2026-09-15).
  Verify identity via `dpkg -V needrestart` and its parent chain before assuming it's part of a
  compromise — but don't skip checking it just because it "looks legitimate" either.

## Security incident — 2026-09-15

An ~8-week-old cryptojacking compromise (`odoo` Postgres role had `SUPERUSER`, letting an
unidentified Odoo-app bug reach OS command execution via `COPY ... FROM PROGRAM`) was found and
contained on this host. Full record: `tpt-dev-scripts/SECURITY-INCIDENT-2026-09-15.md`.
Generalized detection/response steps: `runbooks/postgres-copy-program-compromise.md`. The
`aversan-demo.service` unit mentioned in earlier versions of this file (dead SaaS-16.3 leftover)
was removed as part of this cleanup, along with the `saizen`/`raybright`/`viable`/`17focus`/
`viswox17` retirements reflected in the client table above. **Not fully closed** — see "Still
open" in the incident doc (unrotated `db_password`, `ubuntu` Postgres role still superuser, no
durable egress block yet, and the actual Odoo-side injection point was never identified).
