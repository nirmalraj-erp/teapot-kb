# chakra

**Domain:** Manufacturing / Fabrication / MRP. **Type:** Odoo customization project (bespoke,
not a shared product).
**Repo:** `fabx` (`git@bitbucket.org:teapottechies/fabx.git`, branch `17.0`) — in the
`odoo-17/tpt-ind` workspace. README: "Fabrication, Assembly, MRP, RMA & Repair." Confirmed as
`chakra`'s sole addon repo via `odoo-conf/chakra-17c.conf`'s `addons_path`.
**Host:** `ap-southeast-1` (purchase-heavy backend, worker mode — bumped to 768MB/1GB memory
tier for purchase reporting/import spikes). IP/port-only, no domain/nginx vhost.

## What's known from code

27 modules covering purchase/RFQ workflows (`fabx_rfq_management`, `fabx_purchase_verification`,
`fabx_purchase_revision_extended`, `fabx_po_approval_warning`, `direct_internal_po`), BOM/
manufacturing (`fabx_bom_approve`, `mrp_operating_unit`, `fabx_demand_forecasting`,
`fabx_planning_custom`), inventory/warehouse (`fabx_inventory_overview`,
`warehouse_location_management`, `stock_no_negative`, `fabx_inventory_operating_unit`),
inspection/repair (`fabx_inspection`, `fabx_tpt_repair`, `repair_operating_unit`,
`fabx_maintenance`), and sales-side mirrors of the purchase workflow
(`fabx_sale_verification`, `fabx_sale_revision_extended`). Category convention: `Fabx/<area>`.
Ticket prefix in commit history: `FX-<n>` (e.g. `FX-235`).

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, what `chakra` actually
fabricates/assembles.
