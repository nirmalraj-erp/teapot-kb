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

## Clients (as of last verification 2026-08-17)

| Client | Workload | Status |
|---|---|---|
| `skenmar` | Website only | Running |
| `kships` | Website only | Running |
| `nvt` | POS | Running (worker mode) |
| `mivik` | Standard backend | Running |
| `blaze-xpert-academy` | Standard backend | Running |
| `gadgets` | POS | Running (worker mode) |
| `samaran` | Standard backend | Running |
| `chakra` | Purchase-heavy backend | Running (worker mode) |
| `saizen` | Website only | **Inactive** — stopped 2026-08-17 |
| `sanyo` | Website only | Dormant (configured, not started) |
| `dpc-dental` | Standard backend | Dormant on this host (live deployment is on `eu-central-1` instead — see there) |

Only `skenmar`, `kships`, `nvt`, `mivik`, `blaze-xpert-academy`, and `saizen` (inactive) have a
real domain + nginx vhost; `gadgets`, `samaran`, `chakra` are IP/port-only.

**Authoritative source for ports, memory limits, and current status:**
`tpt-dev-scripts/odoo-conf-summary.md` — check there before touching anything, this file will
drift otherwise. Per-client engagement/business context lives in `clients/`.

## Known quirks

- A live, non-"odoo"-named systemd unit (`aversan-demo.service` — unrelated dead SaaS-16.3
  deployment, inactive since 2025-05-07) is easy to miss when searching unit names for "odoo".
  Check `/proc/<pid>/cgroup` or grep unit file *contents* for the binary path, not names, before
  assuming a process is unsupervised.
- Check for duplicate certbot renewal timers before touching TLS on this host (see
  `runbooks/certbot-duplicate-timers.md`) — confirmed present on the EU host, worth checking here
  too rather than assuming it's clean.
