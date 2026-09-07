# XPOS Tech

**Repo:** `xpos_tech` (`git@bitbucket.org:teapottechies/xpos_tech.git`, branch `17.0`, OCA org
name `xplore_labs`) — in the `odoo-17/tpt-ind` workspace.
**Domain:** POS (Point of Sale).

"POS Pro App for Textile Industry" per its README — a shared library of Point of Sale
customizations, **one repo deployed to two separate production tenants** (own database, own
`addons_path` each — a change here can affect both). 28 modules total in the repo; each client
only installs a subset — see the confirmed per-client lists in `clients/pos/nvt.md` and
`clients/pos/gadgets.md` (verified 2026-09-07 from each client's own module-inventory export,
not inferred from category/theme). Loose thematic grouping, for orientation only:

- Receipt/label printing: `pos_dymo_label_print`, `pos_label_print`, `xpos_barcode_label`,
  `xpos_qr_label`
- POS UX/workflow: `point_of_sale_extend`, `pos_restaurant_ui`, `pos_table_category_filter`,
  `discound_loyalty`, `loyalty_card_merge`
- Stock/reporting guards: `pos_low_stock_badge`, `low_stock_email_chatter_alert`,
  `pos_restrict_product_stock`, `hide_cost_price`
- Invoice/warranty extensions

Most modules use category `XPOS/<Area>`; two (`gadgets_pos_custom_fields`,
`gadgets_receipt_extended`) are scoped to the `gadgets` client specifically under
`Gadgets/<Area>`.

**Not currently installed on either client** (per the 2026-09-07 inventory — dead weight, WIP,
or client-agnostic library code never actually deployed; don't assume these are safe to gut
without checking history first): `contact_number_unique`, `hide_cost_price`, `pos_label_print`,
`pos_restrict_product_stock`, `xpos_barcode_label`, `xpos_pos_booking_receipt`,
`xpos_render_fix`.

**External dependency:** `xpos_settings` (on `nvt`) needs OCA `auth_session_timeout`
(`server-auth` repo), not vendored in this workspace's `xpos_tech` checkout — see the
`addons_path` entries in `servers/client-repo-matrix.md`.

**Known bug (unfixed as of 2026-09-07):** `xpos_mrp_receipt` reads `res.company.mrp_id`, a
field only defined by `xpos_settings` — but `xpos_mrp_receipt`'s manifest doesn't declare
`xpos_settings` as a dependency. Both happen to be installed together on `nvt` today so it isn't
currently biting in production, but installing `xpos_mrp_receipt` anywhere without
`xpos_settings` will crash POS session load with `Invalid field 'mrp_id' on model 'res.company'`.

## Clients

- `clients/pos/gadgets.md`
- `clients/pos/nvt.md`
