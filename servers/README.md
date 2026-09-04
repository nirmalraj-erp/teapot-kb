# servers/

One file per production host: `<region>.md` (e.g. `ap-southeast-1.md`, `eu-central-1.md`).

What belongs in a host file:

- What the host is, which clients it hosts, and its provider/instance details (non-secret —
  e.g. Lightsail instance ID, region, sizing).
- A pointer to `tpt-dev-scripts` (`odoo-conf-summary.md` + its `CLAUDE.md`) as the
  authoritative, regularly-updated source for ports, memory tiers, and per-client config —
  don't fork that detail into this repo, it will drift.
- Operational notes that don't belong in `tpt-dev-scripts` because they're not config: known
  quirks, maintenance windows, capacity ceilings, past incidents worth remembering.

**Never put credentials, SSH keys, or access secrets in this folder** — those live in
`tpt-dev-scripts`' gitignored `.env` (or wherever the team's secrets manager is), never in a
tracked file in this repo.

`client-repo-matrix.md` also lives here: a per-client breakdown, built directly from each
`odoo-conf/*.conf`'s `addons_path`, of which repo(s) and domain each production client actually
uses. Update it whenever a client's `addons_path` changes in a way that affects which repo or
product it depends on.
