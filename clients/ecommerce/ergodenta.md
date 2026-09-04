# ergodenta (ErgoDenta)

**Domain:** E-commerce. **Type:** Odoo customization project — fully bespoke; confirmed via
`odoo-conf/ergodenta-17.conf`'s `addons_path` that it does **not** use the shared
`products/xmart.md` product (unlike `madental`, `dpc-dental`, `mivik`, `skenmar`).
**Repo:** `ergodenta` (`git@bitbucket.org:teapottechies/ergodenta.git`, branch `17.0`, 18
modules) — in the `odoo-17/tpt-eu` workspace.
**Host:** `eu-central-1` (standard backend, threaded, prod). Domains: `ergodenta.com`,
`ergodenta.dk`.

## What's known from code

Vendors `theme_prime`/`droggol_theme_common` themes directly under `ergodenta/theme`. Some
manifests declare dependencies outside this checkout — Odoo Enterprise (`stock_enterprise`,
`hr_contract_reports`) and OCA (`sale_purchase`, `product_variant_sale_price`) — needed on the
`addons_path` if installing those modules.

A staging instance (`ergodenta-test`, `test-odoo.ergodenta.dk`) was retired 2026-09-02 (service,
vhost, cert removed; `17ergodenta_test_demo` DB deliberately kept). Don't recreate staging
without checking why it was retired first.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, what's actually sold/what the store
covers, functional scope of the 18 modules.
