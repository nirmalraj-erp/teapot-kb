# Onboarding a new client's Odoo server

Based on the conventions documented in `tpt-dev-scripts/CLAUDE.md`.

1. **Assign ports.** Pick a free `xmlrpc_port` and `gevent_port` — check
   `tpt-dev-scripts/odoo-conf-summary.md` first; these must be unique *per host* (not globally
   across `ap-southeast-1` and `eu-central-1`).
2. **Copy the config template.** Copy `tpt-dev-scripts/odoo-run-tmpl.conf`, fill in the client
   name and ports. `db_name`/`dbfilter` typically follow `17<client>[.env|_env]` (e.g.
   `17samaran.prod`) or `xpos_<client>_live` for XPOS-branded clients. Set `proxy_mode = True`
   and `without_demo = all` — every production client config has these.
3. **Pick a memory/CPU tier.** Match an existing client's tier rather than inventing new
   numbers — see the two tiers ("standard" vs "light") documented in `tpt-dev-scripts/CLAUDE.md`.
   Remember `limit_memory_soft`/`limit_memory_hard` only do anything if `workers >= 1`; at the
   default `workers = 0` they're inert. Only give a client `workers = 1` if it's POS or another
   workload likely to spike — the servers can't give every instance its own worker pool.
4. **Set `addons_path`.** Absolute paths under `/home/ubuntu/tpt/17.0/` on the deployment host:
   core `odoo/addons` + `odoo/odoo/addons`, then client-specific/shared/OCA repos. Match the
   pattern of a similar existing client rather than inventing a new layout.
5. **Systemd unit.** Copy an existing `tpt-dev-scripts/systemd/odoo17-*.service` (not
   `odoo_template.service`, which is more skeletal) so it inherits `Restart=on-failure` +
   `RestartSec=10` + the `StartLimitBurst=5`/`StartLimitIntervalSec=300` guard.
6. **nginx vhost.** Copy `tpt-dev-scripts/nginx-tmpl.conf` (not an older per-client vhost style)
   — it has the correct `/websocket` proxy handling and the `proxy_intercept_errors on;
   error_page 502 503 504 =503 @maintenance;` maintenance-page fallback (must be the exact `=503`
   form, not bare `=`). Point `proxy_pass` at `localhost:<xmlrpc_port>`. If the client ends up in
   worker mode, `/websocket` must proxy to the separate `gevent_port` instead of the HTTP port.
   Always set `proxy_http_version 1.1;` on that location or the WebSocket upgrade silently fails.
7. **`web.base.url`.** Once the domain is live, set the `ir_config_parameter` to match the real
   `https://` domain, not the IP/port — otherwise the website editor breaks even though normal
   browsing looks fine. If set via raw SQL rather than the ORM, **restart the instance** — the
   running process's in-memory cache won't pick up the change otherwise.
8. **Update `odoo-conf-summary.md`.** This is the single place to check for port collisions —
   update it in the same change that adds the new client conf.
9. **Add a client file here** in `clients/` (see `clients/README.md`) and, once you know which
   host it's on, add it to the relevant `servers/<host>.md` client table.
10. **Confirm the new client's Postgres `db_user` role is NOT superuser.**
    `SELECT rolname, rolsuper, rolcreatedb, rolcreaterole FROM pg_roles WHERE rolname = '<db_user>';`
    — `rolsuper` should be `f`. `CREATEDB`/`CREATEROLE` is enough for Odoo's own database
    management; superuser lets any SQL-injection-shaped bug in the app reach OS command execution
    via `COPY ... TO/FROM PROGRAM`. This is exactly how `ap-southeast-1` was compromised for ~8
    weeks starting 2026-07-20 — see `tpt-dev-scripts/SECURITY-INCIDENT-2026-09-15.md` and
    `runbooks/postgres-copy-program-compromise.md`.
