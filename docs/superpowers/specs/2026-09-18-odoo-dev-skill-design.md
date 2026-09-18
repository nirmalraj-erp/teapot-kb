# odoo-dev skill — design

**Date:** 2026-09-18
**Status:** approved, pending implementation plan

## Problem

TPT's team writes and reviews custom Odoo 17 module code across several client/product repos
(`tpt-ind`, `tpt-eu`, `viswox`), and periodically hits performance and server-resource problems
(slow queries, worker exhaustion, memory pressure). There's currently no shared, canonical
reference for Odoo framework development practices or for the team's own hard-won performance
lessons — that knowledge is either scattered across engagements or lives only in whoever
debugged it last. `teapot-kb` already has a skill for Odoo *ops* (`tpt-odoo-ops`); it has
nothing for Odoo *development*.

## Goal

Add a new skill, `odoo-dev`, alongside the existing `tpt-odoo-ops` skill in `skills/`, giving
Claude Code a reference for:
- General Odoo 17 framework development (ORM, views, module structure, business logic patterns)
- Deep coverage of ORM performance patterns and Odoo's server resource model (workers, memory,
  DB connections) — the two areas explicitly called out as priorities
- TPT's own real findings/incidents in this area, captured and grown the same way
  `tpt-odoo-ops` has grown from real ops incidents

The framework-level content is researched from official Odoo documentation (via Claude's
existing web research tools, written up by hand into markdown) — not scraped by a new script.
This repo stays docs-only; no tooling is added.

## Non-goals

- Not a full re-hosting of Odoo's documentation site. Sections outside the performance/server
  focus (views, security, business logic) stay lean — enough to write correct code, not
  exhaustive.
- Not a duplicate of TPT's actual per-client ops numbers. Memory tiers, ports, and deploy
  procedures stay authoritative in `tpt-dev-scripts` / `tpt-odoo-ops`; `odoo-guidance.md` covers
  only the framework-level *model* of how Odoo consumes server resources (worker sizing formula,
  memory-limit semantics, DB pool vs. worker count, cron/queue_job concurrency) and links out for
  TPT's actual numbers.
- No automated/periodic doc-scraping pipeline. Refreshing the guide for a new Odoo version is a
  manual, Claude-assisted research pass, done when TPT actually starts working in that version —
  not scheduled or automatic.
- No pre-built per-version file structure (`17.md`, `18.md`, ...). Only Odoo 17 is real today;
  18+ is "get ahead of it," not active work. The doc is structured so a per-version split is easy
  later, without building empty scaffolding now.

## Design

### File layout

```
skills/odoo-dev/
  SKILL.md          — trigger + index (tpt-odoo-ops-style)
  odoo-guidance.md  — researched Odoo framework reference
  findings.md       — TPT's own field notes (append-only, never bulk-regenerated)
```

`skills/README.md`'s "Current skills" line is updated to list `odoo-dev` alongside
`tpt-odoo-ops`. No other existing file changes — top-level `CLAUDE.md` structure and
`tpt-odoo-ops` itself are untouched.

### Why guidance and findings are separate files

`odoo-guidance.md` is periodically re-researched/refreshed from official docs as Odoo versions
change — a bulk-rewrite operation. `findings.md` is append-only tribal knowledge (real incidents,
benchmarks, workarounds hit in actual client work) that must never be clobbered by a refresh
pass. Splitting them means "update the guide for Odoo 19" can never accidentally erase a
hard-won finding.

### `odoo-guidance.md` structure

Sections, in order, each ending with a short "Version notes" callout (currently just "Odoo 17
(current)", leaving a marked spot for future version deltas):

1. **Module anatomy** — manifest, models/views/security/data file layout, static assets. Short,
   orientation only.
2. **ORM fundamentals** — recordsets, environment/context, field types, `compute`/`related`/
   `stored`, `@api.depends`/`onchange` vs. `compute`, constraints. Foundation for section 6.
3. **Views & QWeb** — form/list/kanban basics, inheritance via `xpath`, QWeb report templates.
   Kept lean.
4. **Business logic patterns** — wizards, mixins, scheduled actions (`ir.cron`), server actions.
5. **Security** — ACLs, record rules, groups. Enough to write correct code, not a full audit
   guide.
6. **ORM performance** *(deep)* — `search`/`browse`/`read` costs, prefetch behavior, N+1 causes
   and fixes, batch create/write, `sudo()` cost, domain/index interaction, `read_group` vs.
   manual aggregation, `search_count` vs. `len(search())`.
7. **Server architecture & resource utilization** *(deep)* — worker model (prefork vs.
   threaded/dev mode), gevent worker for longpolling, `workers`/`max_cron_threads` sizing
   relative to CPU/RAM, per-worker memory limits (`limit_memory_soft`/`hard`), DB connection pool
   (`db_maxconn`) vs. worker count, cron/queue_job concurrency. Framework-level only — links to
   `tpt-dev-scripts`/`tpt-odoo-ops` for TPT's actual per-client numbers.
8. **Debugging & profiling** — `--log-sql`/query counting, `odoo shell`, profiler/`py-spy`,
   reading `pg_stat_statements`. Makes sections 6–7 actionable.

### `SKILL.md`

Same shape as `tpt-odoo-ops`:
- Frontmatter `description` triggers on both dev-time work (writing/reviewing Odoo module code —
  ORM, views, business logic — in the `tpt-ind`/`tpt-eu`/`viswox` repos) and troubleshooting
  (slowness, timeouts, high memory/CPU, worker exhaustion).
- Body: "Where things live" pointing to `odoo-guidance.md` and `findings.md`; an explicit note
  that TPT's actual per-client memory tiers/ports/ops procedures live in
  `tpt-dev-scripts`/`tpt-odoo-ops`, not duplicated here.
- A "Refreshing this guide for a new Odoo version" procedure: when TPT starts real 18+ work,
  research that version's release notes/ORM changelog, add a "Version notes" delta to affected
  topic sections rather than rewriting the baseline, and only split into a per-version file if a
  topic's delta gets large enough to warrant it.

### `findings.md`

Starts with a short header (purpose: TPT's own hard-won lessons; append-only; never bulk-
regenerated) and one entry template:

```
## <short title> (<client/product>, <date>)
**Symptom:** what was observed
**Root cause:** what was actually wrong
**Fix / takeaway:** what to do differently
```

Starts with zero fabricated entries — real entries get added as the team hits them.

## Content generation approach

`odoo-guidance.md`'s researched sections (1–8) are written by researching official Odoo 17
developer documentation (via web research tools) and synthesizing findings into the markdown
by hand — this is real content, not placeholder headers. The implementation plan should treat
each deep section (6, 7, 8) as requiring its own research pass given their priority and depth;
sections 1–5 can be written more directly given they're intentionally lean.

## Testing / validation

This is a documentation-only change — no code, no build/test tooling applies. Validation is
review: content should be technically accurate for Odoo 17, cross-references to
`tpt-dev-scripts`/`tpt-odoo-ops` should resolve to real files/sections, and `skills/README.md`'s
index should stay accurate.
