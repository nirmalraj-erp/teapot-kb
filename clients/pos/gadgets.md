# gadgets

**Domain:** POS. **Product:** `products/xpos-tech.md` (shared repo with `nvt` — own database,
own `addons_path`, separate production tenant; a change to the shared repo can affect both).
**Host:** `ap-southeast-1` (worker mode — `workers = 1` for real memory enforcement).

Client-specific modules beyond the shared library: `gadgets_pos_custom_fields`,
`gadgets_receipt_extended` (category `Gadgets/<Area>`).

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, retail vertical/product line.
