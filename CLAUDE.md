# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`teapot-kb` is the internal knowledge base for **Teapot Techies Pvt Ltd (TPT)**, an Odoo
development shop that builds client Odoo 17 projects and its own product, **VisWoX ERP**
(a construction-industry ERP). This repo is for the team of developers and functional
consultants working across TPT's client engagements — it's meant to hold the knowledge that
isn't already captured in each project's own codebase: client/engagement context, server and
access details, cross-project decisions, and anything else the team needs to stay in sync.

**As of 2026-09-04 this repo is newly created** — the folder structure exists (see below) but
is not yet populated with content. There is no application code, so no build/lint/test tooling
applies here — "work" in this repo means writing and organizing documentation, not writing or
running code.

## Structure

TPT's work splits into two kinds of thing, and the structure below mirrors that:

- **Products** — one codebase, multiple deployments, TPT owns the roadmap: VisWoX (construction
  ERP) and XPOS Tech (POS library). Documented in `products/`.
- **Customization projects** — bespoke, single-client Odoo repos (`motox`, `xperts_academy`,
  `ergodenta`, `madental`, and others). Documented in `clients/`, grouped into domain subfolders.

- `products/` — one file per shared product (what it is, editions/variants, architecture,
  which clients run it): VisWoX, XPOS Tech, XMart. See `products/README.md`.
- `clients/` — one file per client, grouped by domain: `pos/`, `ecommerce/`, `academy/`,
  `fleet/`, `manufacturing/`, `website/` (no custom repo at all). See `clients/README.md`.
- `servers/` — one file per production host (what it hosts, non-secret operational notes),
  plus `client-repo-matrix.md` — the authoritative client→repo→domain mapping, built directly
  from each `odoo-conf/*.conf`'s `addons_path` (don't guess domain from workspace location;
  confirm it from `addons_path` the way that file does). See `servers/README.md`. Ports/config
  stay authoritative in `tpt-dev-scripts`, not here.
- `projects/` — cross-cutting initiatives not scoped to one client (e.g. multi-client
  migrations). See `projects/README.md`.
- `runbooks/` — step-by-step operational procedures ("what to do when X"). See
  `runbooks/README.md`.
- `skills/` — canonical copies of Claude Code skills the team has written for TPT work
  (`<skill-name>/SKILL.md`). Claude Code only loads skills from a project's own
  `.claude/skills/` or your user-level `~/.claude/skills/`, not from here directly — see
  `skills/README.md` for how to actually put one to use.

Each folder's `README.md` is the authority on what belongs in it — check there before adding
a file in the wrong place.

**Never commit credentials, API keys, or SSH access secrets into this repo.** Every sibling
project below already keeps live secrets out of git (e.g. `tpt-dev-scripts`' server SSH details
live in a gitignored `.env`, never in tracked files) — follow the same rule here even though this
is "just docs."

## The Teapot Techies project landscape

TPT's work is split across several separate repositories, each already documented with its own
`CLAUDE.md`. Read the relevant one before working in that codebase — this file should stay a
high-level map between them, not duplicate their content.

| Repo | Path | What it is |
|---|---|---|
| `odoo-17/tpt-ind` | `~/PycharmProjects/odoo-17/tpt-ind` | A workspace folder (not itself a git repo) bundling vanilla Odoo 17 core plus client/product addon repos: **`motox`** (client `samaran` — fleet), **`xperts_academy`** (client `blaze-xpert-academy` — academy), **`xpos_tech`** (POS product — `gadgets`, `nvt`), **`fabx`** (client `chakra` — manufacturing/MRP), **`xmart`** (e-commerce product — `dpc-dental`, `mivik`, `skenmar`, plus `madental` in `tpt-eu`). Its own `CLAUDE.md` was written before `fabx`/`xmart` were cloned and doesn't mention them yet |
| `odoo-17/tpt-eu` | `~/PycharmProjects/odoo-17/tpt-eu` | Same kind of workspace folder, bundling Odoo 17 core plus two client addon repos: **`ergodenta`** (client ErgoDenta) and **`madental`** (client Madental) — both e-commerce clients, hosted in `eu-central-1` |
| `tpt-dev-scripts` | `~/PycharmProjects/tpt-dev-scripts` | Ops/config-only repo: per-client Odoo server configs (`odoo-conf/`), nginx reverse-proxy vhosts, systemd units, and the port/memory allocation table (`odoo-conf-summary.md`) for both production hosts |
| `viswox` | `~/PycharmProjects/viswox` | TPT's own product: **VisWoX ERP**, a construction-industry Odoo 17 product covering planning → procurement → site execution → finance, shipped in Standard/Premium/Pilot commercial editions |
| `viswox-static-website` | `~/PycharmProjects/viswox-static-website` | Two unrelated projects: VisWoX's public marketing site (static HTML/CSS/JS, live at viswoxerp.com) and a separate `seo-monitor` bot that audits it from the outside |

`tpt-ind` and `tpt-eu` are plain folders holding several independent git checkouts (Odoo core
plus one repo per client) — always `cd` into the specific client repo before running `git`
there. The other three are each a single git repo.

## Infrastructure at a glance

Client Odoo 17 instances run on two production Lightsail hosts, both managed through
`tpt-dev-scripts`:

- **`ap-southeast-1`** — primary host; clients include `skenmar`, `kships`, `nvt`, `mivik`,
  `blaze-xpert-academy`, `gadgets`, `samaran`, `chakra`, plus `dpc-dental` (dormant there).
  `saizen` was fully retired 2026-09-15 (see `servers/ap-southeast-1.md`).
- **`eu-central-1`** (`3.125.249.119`) — hosts `ergodenta`, `madental`, and the live
  `dpc-dental` instance.

Port numbers and memory tiers are only unique *per host*, not globally across both. SSH access,
port allocation, and current client status are tracked in `tpt-dev-scripts` (its
`odoo-conf-summary.md` and `CLAUDE.md` are the authoritative, regularly-updated source) — link to
that rather than copying details here, since they drift.

## Working in this repo

Since there's little content yet, use judgment grounded in the stated purpose —
collaboration and staying in sync across dev + functional consultants — when deciding what
goes where. Confirm any new top-level structure beyond the four folders above with the user
first, and keep this file's map in sync with whatever exists.
