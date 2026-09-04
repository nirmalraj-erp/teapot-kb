# clients/

One file per **customization-project** client (bespoke, single-client Odoo work) — as opposed
to `products/`, which covers TPT codebases shared across multiple clients (VisWoX, XPOS Tech).
Grouped into domain subfolders:

- `pos/` — Point of Sale clients built on the shared `products/xpos-tech.md` product
- `ecommerce/` — e-commerce/storefront clients, some bespoke, some built on the shared
  `products/xmart.md` product (check each file — see `servers/client-repo-matrix.md` for which)
- `academy/` — education/academy-management customization projects
- `fleet/` — fleet/rental-management customization projects
- `manufacturing/` — fabrication/MRP/purchase-workflow customization projects
- `website/` — clients with **no** client-specific or shared-product addon repo at all, just a
  theme (confirmed via `addons_path` — see `servers/client-repo-matrix.md`). Move a client out
  of here the moment a real repo shows up in its `addons_path`.

If a client's addon repo hasn't been located/cloned yet, its domain is unknown — check
`odoo-conf/<client>.conf`'s `addons_path` once cloned (this is how every domain above was
actually confirmed, not guessed) and file it in the right folder rather than leaving it
unclassified.

Filename: `<client-slug>.md`, matching the slug used in `odoo-conf/*.conf` and the client's
addon repo where one exists (e.g. `ergodenta.md`, `samaran.md`).

What belongs in a client file:

- Domain and, if applicable, which shared product it's built on (link to `products/`).
- Which repo it lives in and which host it's deployed on — link out to that repo's own
  `CLAUDE.md` and to `servers/` rather than duplicating deploy detail here.
- Key contacts (client-side and internal), and functional decisions/history that explain
  *why* things are the way they are — the kind of context that isn't visible from reading
  code or config alone.
- Support/escalation notes specific to this client.

What doesn't belong here: server config, ports, credentials — that's `servers/` (or the
gitignored `.env` in `tpt-dev-scripts`, never this repo).
