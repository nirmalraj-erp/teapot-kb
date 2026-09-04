# VisWoX

**Repo:** `viswox` (`~/PycharmProjects/viswox`, custom code in the nested `viswox/` addon dir).
**Domain:** Construction ERP — TPT's own product (not a client customization project).

Covers the full construction project lifecycle: planning → procurement → site execution →
finance. Ships in three commercial editions, each module's manifest `"label"` field recording
which edition(s) it belongs to (`custom_res_config_edition` surfaces the active tier in
Settings):

| Edition | Scope | Notes |
|---|---|---|
| **Standard** | Full product | Capped by `viswox_license_limits` — 5 non-admin users / 10GB storage by default, adjustable per customer, stacked with capacity add-on packs |
| **Premium** | Standard + India rate-intelligence, smart BOM auto-import, in-system spreadsheet BOQ/BOM drafting | Same seat/storage defaults as Standard |
| **Pilot** | Standard + `viswox_pilot_license` | Shared trial database hosting *multiple clients as separate `res.company` records* — 30-day expiry, 1 project/50 tasks/3 users/1GB per company, whole DB capped at 5 concurrent active companies |

## Core business flow

1. **Project Creation** → auto-creates an Inventory Location for the site
2. **BOQ** (`viswox_project_costing`) → Bill of Quantities with unit rates/quantities → sets budget
3. **BOM Generation** → manual (`viswox_bom_boq_manual_import`) or smart
   (`viswox_bom_boq_intel_import`) — derives execution materials from BOQ
4. **Indent Request** (`viswox_procurement`, via OCA `purchase_request`) → material requisition
5. **Procurement** → Direct Stocking (warehouse→site transfer) or Drop Shipping (PO→vendor→site)
6. **Consumption Tracking** → `viswox.project.bom.line` tracks estimated vs. consumed;
   `project_dashboard` shows live Budget vs. Actual
7. **Change Orders/Revisions** (`viswox_boq_versioning`) → tracked BOQ revisions with
   reject/approve wizard flow
8. **Expenses** (`viswox_project_expense`) → petty cash/HR expense tied to project cost rollup

`viswox_project_costing` is the most critical module — a direct dependency of 12+ other VisWoX
modules; it owns the BOQ/BOM models and project budget tracking.

## Development process

Non-trivial features are documented spec-then-plan under `viswox/docs/superpowers/`: a dated
spec (`specs/YYYY-MM-DD-<slug>-design.md`, the why/requirements) followed by a dated plan
(`plans/YYYY-MM-DD-<slug>.md`, the implementation steps). Check there for rationale on past
product decisions before starting related work — don't duplicate that history here.

## Clients

VisWoX doesn't appear in `tpt-dev-scripts/odoo-conf-summary.md` the way customization-project
clients do — Standard/Premium are one-company-per-database, Pilot hosts multiple clients as
`res.company` records within one shared database. TODO: document where VisWoX's own production
database(s) are hosted and which real customers are on it, once that's confirmed.
