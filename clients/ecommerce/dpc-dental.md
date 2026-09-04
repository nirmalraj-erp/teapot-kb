# dpc-dental

**Domain:** E-commerce (same market as `ergodenta`/`madental`).
**Type:** Runs on the shared `products/xmart.md` product, no bespoke repo of its own — confirmed
via both `odoo-conf/dpc-dental-17c.conf` and `dpc-dental-eu-17c.conf`'s `addons_path`
(`theme`, `design-themes`, `xmart`, `oca-repo/server-brand`, `oca-repo/server-tools`,
`oca-repo/server-auth` — identical on both hosts, no client-specific repo).
**Host:** Live on `eu-central-1` (`dpc-dental-eu-17c.conf`); dormant on `ap-southeast-1`
(`dpc-dental-17c.conf`) — deliberately split into separate config/systemd pairs per host since
2026-09-02, don't re-merge them.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, what's actually sold/what the store
covers, why it runs dormant on `ap-southeast-1` at all if the live deployment is on
`eu-central-1`. Fill in repo details once cloned.
