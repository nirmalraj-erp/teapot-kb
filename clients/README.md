# clients/

One file per **customization-project** client (bespoke, single-client Odoo work) — as opposed
to `products/`, which covers TPT codebases shared across multiple clients (VisWoX, XPOS Tech).
Grouped into domain subfolders:

- `pos/` — Point of Sale clients built on the shared `products/xpos-tech.md` product
- `ecommerce/` — e-commerce/storefront customization projects
- `academy/` — education/academy-management customization projects
- `fleet/` — fleet/rental-management customization projects
- `unclassified/` — clients whose addon repo hasn't been located/cloned yet, so their domain
  is unknown. Re-tag into a proper domain folder once the repo is cloned and reviewed — see
  each file's TODO.

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
