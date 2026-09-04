# products/

TPT-owned codebases deployed to **multiple** clients from one shared repo, as distinct from
`clients/` (bespoke, single-client customization projects). One file per product.

What belongs in a product file: what it is, its repo(s), its edition/variant model if any, its
core architecture/business flow, and which clients run it (cross-link to `clients/`). Product
roadmap/initiative work that spans multiple clients belongs in `projects/`, not here — this file
should describe what the product *is*, not what's currently being built for it.

Current products: `viswox.md` (construction ERP), `xpos-tech.md` (POS customization library).
