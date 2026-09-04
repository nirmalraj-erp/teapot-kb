# XPOS Tech

**Repo:** `xpos_tech` (`git@bitbucket.org:teapottechies/xpos_tech.git`, branch `17.0`, OCA org
name `xplore_labs`) — in the `odoo-17/tpt-ind` workspace.
**Domain:** POS (Point of Sale).

"POS Pro App for Textile Industry" per its README — a shared library of Point of Sale
customizations, **one repo deployed to two separate production tenants** (own database, own
`addons_path` each — a change here can affect both):

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

## Clients

- `clients/pos/gadgets.md`
- `clients/pos/nvt.md`
