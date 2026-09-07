# gadgets

**Domain:** POS. **Product:** `products/xpos-tech.md` (shared repo with `nvt` — own database,
own `addons_path`, separate production tenant; a change to the shared repo can affect both).
**Host:** `ap-southeast-1` (worker mode — `workers = 1` for real memory enforcement).

Client-specific modules beyond the shared library: `gadgets_pos_custom_fields`,
`gadgets_receipt_extended` (category `Gadgets/<Area>`).

## Installed modules

Verified 2026-09-07 from the client's own module-inventory export (Apps list, "Installed"
filter) — this is the actual production install list, not inferred from `addons_path` or
category. 10 `xpos_tech` modules + 1 module from outside `xpos_tech`:

| Technical name | Display name |
|---|---|
| `gadgets_pos_custom_fields` | Gadgets Pos Custom Fields |
| `gadgets_receipt_extended` | Gadgets Receipt Customization |
| `product_pos_and_invoice_warranty` | Product Pos and invoice warranty |
| `pos_dymo_label_print` | POS DYMO Label Print |
| `pos_low_stock_badge` | Product Low Stock Badge Alert |
| `pos_table_category_filter` | POS Table Category Filter |
| `pos_reporting_access` | POS Reporting Access |
| `low_stock_email_chatter_alert` | Low Stock Email & Chatter Alert |
| `pos_restaurant_ui` | Restaurant POS UI Update |
| `xpos_pos_invoice_actions` | POS Invoice Actions (Print / Email) |
| `account_report_hide_empty_taxes` | Hide Empty Taxes Column in Invoice Report |

**`account_report_hide_empty_taxes` is a cross-repo surprise.** Its technical name and manifest
match a module that lives in the `xperts_academy` repo in this workspace (an India-localization
invoicing tweak, normally associated with the `blaze-xpert-academy` client, not POS). But
`gadgets`' production `addons_path` (`tpt-dev-scripts/odoo-conf/gadgets-17c.conf`) does **not**
include `xperts_academy` — it includes a `tpt_addons` path entry instead, which
`servers/client-repo-matrix.md` already flags as "not located/reviewed yet". An old local backup
checkout (`~/Documents/backup/PycharmProjects/tpt/tpt-addons/`) has a `tpt_addons/` folder that
itself nests a copy of `xperts_academy/account_report_hide_empty_taxes` — consistent with
`tpt_addons` being a grab-bag directory of repos on the server, but **not confirmed against the
live server's actual directory layout**. TODO: SSH in and check
`/home/ubuntu/tpt/17.0/tpt_addons/` directly to confirm whether it's this same module or a
separate copy, and whether it's meant to be pulled in one level up in `addons_path`.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, retail vertical/product line.
