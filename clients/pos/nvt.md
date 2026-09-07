# nvt

**Domain:** POS. **Product:** `products/xpos-tech.md` (shared repo with `gadgets` — own
database, own `addons_path`, separate production tenant).
**Host:** `ap-southeast-1` (worker mode).

## Installed modules

Verified 2026-09-07 from the client's own module-inventory export (Apps list, "Installed"
filter) — this is the actual production install list, not inferred from `addons_path` or
category. 11 `xpos_tech` modules + 1 OCA module:

| Technical name | Display name |
|---|---|
| `point_of_sale_extend` | Custom POS |
| `pos_discount_fixed_limit` | POS Global Discounts |
| `xpos_global_remove` | POS Remove Product |
| `xpos_order_line_salesperson` | POS Order Line Sales Person |
| `xpos_mrp_receipt` | POS Custom Receipt |
| `xpos_qr_label` | XPOS QR Code in label |
| `xpos_settings` | POS Settings |
| `loyalty_card_merge` | Loyalty Card Merge |
| `xpos_draft_invoice` | XPOS Draft Invoice |
| `xpos_product_sequence` | POS Sequence Generator |
| `discound_loyalty` | POS Discount Loyalty |
| `auth_session_timeout` | — (OCA `server-auth`, hard dependency of `xpos_settings`) |

Note: `xpos_mrp_receipt` ("POS Custom Receipt") is distinct from `xpos_pos_booking_receipt`
("POS Booking Receipt") — the latter is **not** installed on `nvt` (or `gadgets`) currently, see
`products/xpos-tech.md`'s "not currently deployed anywhere" list.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, retail vertical/product line.
