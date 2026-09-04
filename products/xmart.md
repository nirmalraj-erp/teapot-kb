# XMart

**Repo:** `xmart` (`git@bitbucket.org:teapottechies/xmart.git`, branch `17.0`) — in the
`odoo-17/tpt-ind` workspace. README: "Centralized Repo for E-commerce related modules."
**Domain:** E-commerce.

A shared library of website/e-commerce customizations, layered onto core `website`/
`website_sale` — not a standalone product with its own edition system (unlike VisWoX), closer
in shape to XPOS Tech: one repo, several tenants.

Modules seen: `auth_signup_email_format`, `auth_signup_verify_email_fix` (signup/auth tweaks),
`bulk_product_image_import`, `website_custom_checkout_button` (renames Checkout to "Send
Enquiry" — a B2B-enquiry pattern rather than direct online payment), `website_form_rounded_corner`,
`website_internal_reference_display`, `website_phone_validation`, `website_product_overview_page`,
`website_snippet_header`, `website_theme_prime_snippet_footer` (pairs with the `theme_prime`
theme also used standalone by `ergodenta`/`madental`).

## Confirmed from `odoo-conf/*.conf` `addons_path`

- **Pure XMart deployments** (no client-specific repo at all): `dpc-dental` (both
  `dpc-dental-17c.conf` and `dpc-dental-eu-17c.conf`), `mivik-17c.conf`, `skenmar-17c.conf`
- **Layered on top of a bespoke client repo**: `madental-17.conf` (addons_path has both `xmart`
  and `madental`)
- **Confirmed NOT used**: `ergodenta-17.conf` — fully bespoke, no `xmart` in its `addons_path`

See `servers/client-repo-matrix.md` for the full per-client breakdown.

## Clients

- `clients/ecommerce/dpc-dental.md`
- `clients/ecommerce/mivik.md`
- `clients/ecommerce/skenmar.md`
- `clients/ecommerce/madental.md` (partial — also runs its own bespoke `madental` repo)
