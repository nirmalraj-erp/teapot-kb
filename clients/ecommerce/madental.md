# madental (Madental)

**Domain:** E-commerce. **Type:** Bespoke Odoo customization project — but also layers in the
shared `products/xmart.md` product (unlike `ergodenta`, which is pure bespoke).
**Repo:** `madental` (`git@bitbucket.org:teapottechies/madental.git`, branch `17.0`, 37 modules)
— in the `odoo-17/tpt-eu` workspace. Confirmed via `odoo-conf/madental-17.conf`'s `addons_path`,
which includes both `xmart` and `madental` (plus `enterprise`, `design-themes`, `theme`,
`server-auth`, `website`, `social`, `tpt_addons`).
**Host:** `eu-central-1` (standard backend, threaded, prod). Domain: `madental.dk`.

## What's known from code

Same market as `ergodenta`, separate codebase/tenant. Vendors `theme_prime`/
`droggol_theme_common` themes directly under `madental/theme`. Some manifests declare
dependencies outside this checkout — Odoo Enterprise (`stock_enterprise`, `hr_contract_reports`
in `mad_report_pivot_view_default`) and OCA (`sale_purchase`, `product_variant_sale_price`) —
needed on the `addons_path` if installing those modules.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, what's actually sold/what the store
covers, functional scope of the 37 modules (largest client module count of the portfolio —
worth understanding why).
