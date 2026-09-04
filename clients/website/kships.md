# kships

**Domain:** Website (no custom code). **Type:** N/A — no client-specific addon repo.
**Repo:** None — confirmed via `odoo-conf/kships-17c.conf`'s `addons_path`, which contains only
`design-themes`, `server-brand`, and `oca-repo/server-tools`. No bespoke or shared
domain-product repo (`xmart`, `xpos_tech`, etc.) is referenced.
**Host:** `ap-southeast-1` ("website only" workload, threaded). Has a real domain + nginx vhost.

Runs on a purchased/shared theme with no custom modules at all — whatever it needs is either
built into core Odoo (`website`, possibly `website_sale`) or the theme itself. Re-check this
file if a client-specific repo for `kships` ever gets added to `addons_path`.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, what the site actually is/sells.
