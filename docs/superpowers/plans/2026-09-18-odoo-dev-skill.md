# odoo-dev Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a new `odoo-dev` skill to `skills/` — an Odoo 17 framework development reference (`odoo-guidance.md`) plus a TPT field-notes log (`findings.md`) — following the same shape as the existing `tpt-odoo-ops` skill.

**Architecture:** Three new files under `skills/odoo-dev/` (`SKILL.md`, `odoo-guidance.md`, `findings.md`) plus a one-line edit to `skills/README.md`'s index. No code, no tooling — this is a docs-only repo and stays that way. `odoo-guidance.md` content is researched from official Odoo 17 developer documentation and written by hand; `findings.md` starts as an empty, templated log (no fabricated entries).

**Tech Stack:** Markdown, Claude Code skill format (YAML frontmatter + Markdown body). Research via WebFetch/WebSearch against `odoo.com/documentation/17.0`.

**Spec:** `docs/superpowers/specs/2026-09-18-odoo-dev-skill-design.md`

## Global Constraints

- Docs-only repo: no scripts, no tooling, no code files (per this repo's `CLAUDE.md` and the spec's non-goals).
- `odoo-guidance.md` never duplicates TPT's actual per-client ops numbers (ports, memory tiers, deploy steps) — those stay authoritative in `tpt-dev-scripts`/`tpt-odoo-ops`; link out instead.
- `findings.md` is append-only and starts with **zero** fabricated entries — only real, user-supplied incidents get written in, ever.
- Every major section in `odoo-guidance.md` ends with a short "**Version notes:**" line reading "Odoo 17 (current)." — a marked spot for future version deltas, per the spec's version-refresh procedure.
- Baseline is Odoo 17 only; no per-version files (`17.md`, `18.md`, ...) get created in this plan.
- No secrets, credentials, or SSH details get written anywhere in this repo (standing rule from this repo's `CLAUDE.md`).

---

## File Map

```
skills/odoo-dev/
  SKILL.md          — Task 1
  odoo-guidance.md  — Tasks 2–6 (sections appended incrementally)
  findings.md       — Task 7
skills/README.md    — Task 1 (index line edit)
```

---

### Task 1: Skill scaffolding — `SKILL.md` + index update

**Files:**
- Create: `skills/odoo-dev/SKILL.md`
- Modify: `skills/README.md` (its "Current skills" line)

**Interfaces:**
- Produces: `skills/odoo-dev/SKILL.md`, which later tasks' content in `odoo-guidance.md`/`findings.md` must match by filename exactly (no renaming later).

- [ ] **Step 1: Create `skills/odoo-dev/SKILL.md`**

```markdown
---
name: odoo-dev
description: Reference for developing and troubleshooting Odoo 17 custom modules across TPT's client and product repos (tpt-ind, tpt-eu, viswox) — ORM usage, views, module structure, business logic patterns, and (in depth) ORM performance and server resource utilization. Use whenever writing or reviewing Odoo module code, or when diagnosing slowness, timeouts, high memory/CPU, or worker exhaustion on an Odoo instance.
---

# Odoo 17 development reference

## Where things live

- `odoo-guidance.md` — the framework reference: module structure, ORM fundamentals, views,
  business logic patterns, security, and (in depth) ORM performance and server resource
  utilization. Framework-level only.
- `findings.md` — TPT's own field notes: real, dated incidents from client/product work.
  Append-only; check here for precedent before assuming a problem is new.
- TPT's actual per-client memory tiers, ports, and deploy/ops procedures are **not** duplicated
  here — see `tpt-dev-scripts` (`odoo-conf-summary.md`, its `CLAUDE.md`) and the `tpt-odoo-ops`
  skill for those. This skill covers how Odoo's framework consumes server resources in general;
  those repos cover TPT's actual numbers.

## Refreshing this guide for a new Odoo version

Everything here is Odoo 17 baseline today. When TPT starts real work on Odoo 18 or later:

1. Research that version's release notes and ORM/API changelog against the affected topic in
   `odoo-guidance.md`.
2. Add a delta under that topic's existing "**Version notes:**" line — don't rewrite the Odoo 17
   baseline unless it's actually being retired from TPT's work.
3. Only split a topic into its own per-version file if that topic's delta grows large enough
   that inline notes get hard to follow. Don't pre-build empty version files before that's true.

## Adding a finding

When you or the team hits a real, measured performance/server problem, add it to `findings.md`
using the template at the top of that file. Don't add hypothetical or "could happen" entries —
only things actually observed. If it's a specific instance of a general pattern already
documented in `odoo-guidance.md`'s ORM performance section, cross-reference that section instead
of re-explaining the pattern.
```

- [ ] **Step 2: Read `skills/README.md`, then update its "Current skills" line**

Read `skills/README.md` first (required before editing). Change:

```markdown
Current skills: `tpt-odoo-ops` (TPT Odoo production deploy/ops runbook).
```

to:

```markdown
Current skills: `tpt-odoo-ops` (TPT Odoo production deploy/ops runbook), `odoo-dev` (Odoo 17
development reference — framework, ORM performance, server resource utilization).
```

- [ ] **Step 3: Verify**

Run: `test -f skills/odoo-dev/SKILL.md && grep -c "^---$" skills/odoo-dev/SKILL.md`
Expected: file exists; the frontmatter delimiter count is `2`.

Run: `grep "odoo-dev" skills/README.md`
Expected: one match, the updated "Current skills" line.

- [ ] **Step 4: Commit**

```bash
git add skills/odoo-dev/SKILL.md skills/README.md
git commit -m "Scaffold odoo-dev skill

Adds SKILL.md for the new Odoo 17 development reference skill and
lists it in skills/README.md's index."
```

---

### Task 2: `odoo-guidance.md` skeleton + Section 1 (Module anatomy) + Section 2 (ORM fundamentals)

**Files:**
- Create: `skills/odoo-dev/odoo-guidance.md`

**Interfaces:**
- Consumes: nothing from other tasks (file is created fresh here).
- Produces: `skills/odoo-dev/odoo-guidance.md` with a top-level header and Sections 1–2. Tasks 3–6 append further `##` sections to this same file, in numeric order, each followed by a `**Version notes:**` line — later tasks must match this heading style (`## N. <Title>`) exactly.

- [ ] **Step 1: Research**

Fetch and read (WebFetch) the official Odoo 17 developer documentation:
- `https://www.odoo.com/documentation/17.0/developer/reference/backend/module.html` (module structure/manifest)
- `https://www.odoo.com/documentation/17.0/developer/reference/backend/orm.html` (ORM fundamentals)
- `https://www.odoo.com/documentation/17.0/developer/tutorials/server_framework_101.html` (if reachable — general orientation)

If any URL 404s or the doc has moved, use WebSearch for `"odoo 17 developer documentation" <topic>` and use the top official `odoo.com/documentation/17.0/...` result instead. Do not substitute a third-party blog as the primary source — official docs only, third-party sources may only supplement.

- [ ] **Step 2: Write the file header and Section 1**

Create `skills/odoo-dev/odoo-guidance.md` starting with:

```markdown
# Odoo 17 development guidance

Framework-level reference for writing and troubleshooting Odoo 17 custom modules. Baseline is
Odoo 17; see `SKILL.md`'s "Refreshing this guide" procedure for how later versions get added.
This is deliberately lean outside the performance/server sections (6–7) — enough to write
correct code, not an exhaustive restatement of Odoo's own documentation.

## 1. Module anatomy

A module is a directory containing `__manifest__.py` plus the standard subfolders:

- `__manifest__.py` — name, version (`17.0.x.y.z`), `depends`, `data` (XML/CSV files loaded at
  install/update, in listed order), `installable`, `application`.
- `models/` — Python model definitions (one file per model is the convention; `__init__.py`
  imports each).
- `views/` — XML: form/list/kanban/search views, menu items, actions. Loaded via `data` in the
  manifest, not auto-discovered.
- `security/ir.model.access.csv` — per-model, per-group CRUD grants. Required for every new
  model or the model is invisible/inaccessible even to admins.
- `security/*.xml` — record rules and groups, when access needs to be row-level or a new group
  is introduced.
- `data/` — demo/seed data (`noupdate="1"` for data that shouldn't be reset on module update).
- `static/description/` — icon and README shown in the Apps list; `static/src/` — JS/CSS/QWeb
  frontend assets, declared via an `assets` key in the manifest (Odoo 17; pre-16 used a
  `web.assets_backend` XML template instead — not relevant to 17 but worth knowing if reading
  older module code).
- `report/` — QWeb report templates and their Python report actions.

**Version notes:** Odoo 17 (current).

## 2. ORM fundamentals

- **Recordsets** are the ORM's core abstraction: every value returned by `search`/`browse` is an
  ordered, immutable *set* of records of one model — even a "single" record is a recordset of
  size 1. Iterate (`for rec in recordset:`) rather than assume singleton, unless a method is
  explicitly documented as requiring `self.ensure_one()`.
- **`env`** (`self.env`) carries the current user, context, and cursor. `self.env['model.name']`
  gets an empty recordset of that model to call `search`/`create`/`browse` on.
- **`with_context(...)`, `sudo()`, `with_user(...)`** each return a *new* recordset bound to a
  modified environment — they don't mutate `self`. `sudo()` bypasses ACLs/record rules, not
  field-level `groups=` restrictions.
- **Field types**: `Char`, `Text`, `Integer`, `Float`, `Boolean`, `Date`/`Datetime`, `Selection`,
  `Many2one`, `One2many`, `Many2many`, `Binary`. `Many2one` stores a foreign key column;
  `One2many` is a virtual reverse-lookup (no column on this model — needs an `inverse_name`
  pointing at the `Many2one` on the other side); `Many2many` uses a hidden relation table.
- **Computed fields** (`compute=`): declare with `@api.depends('field1', 'field2')` above the
  compute method. The method assigns `record.field = value` for each record in `self` — it must
  handle the whole recordset, not just one record (see Section 6 for why looping `write()` inside
  a compute is expensive). `store=True` persists the value and lets it be searched/sorted on, at
  the cost of recomputation on every dependency change; `store=False` (default) recomputes on
  every read and can't be searched without also defining `search=`.
- **`related=` fields** are a thin compute that reads through a relation path (e.g.
  `related='partner_id.email'`) — implicitly `store=False` unless `store=True` is set explicitly.
- **`onchange` vs. `compute`**: `@api.onchange` only fires in the UI, on a *specific* field edit,
  before save — it never runs on `write()`/`create()` calls made from code (imports, other
  modules, `write()` in a loop). A value that must always be correct regardless of how the
  record was written needs a stored `compute`, not an `onchange`.
- **`@api.constrains`** fires on `create()`/`write()` when a listed field is part of the call,
  and must raise `ValidationError` to reject the data — it is a *check*, not a place to derive or
  write values (see Section 6 for the cost and correctness problems of using it to write).
- **`_sql_constraints`** — a class-level list of `(name, sql, message)` tuples, enforced by
  Postgres itself; cheaper than a Python constraint when the check is expressible in SQL (e.g.
  `UNIQUE`, `CHECK`).

**Version notes:** Odoo 17 (current).
```

- [ ] **Step 3: Verify**

Run: `grep -E "^## [0-9]" skills/odoo-dev/odoo-guidance.md`
Expected: two lines, `## 1. Module anatomy` and `## 2. ORM fundamentals`.

Run: `grep -c "Version notes:" skills/odoo-dev/odoo-guidance.md`
Expected: `2`.

- [ ] **Step 4: Commit**

```bash
git add skills/odoo-dev/odoo-guidance.md
git commit -m "Add odoo-guidance.md sections 1-2 (module anatomy, ORM fundamentals)"
```

---

### Task 3: Section 3 (Views & QWeb) + Section 4 (Business logic patterns) + Section 5 (Security)

**Files:**
- Modify: `skills/odoo-dev/odoo-guidance.md` (append)

**Interfaces:**
- Consumes: existing file from Task 2 — append after Section 2, do not alter Sections 1–2.
- Produces: Sections 3–5, same heading style (`## N. <Title>` + trailing `**Version notes:**`).

- [ ] **Step 1: Research**

WebFetch:
- `https://www.odoo.com/documentation/17.0/developer/reference/backend/views.html`
- `https://www.odoo.com/documentation/17.0/developer/reference/backend/actions.html`
- `https://www.odoo.com/documentation/17.0/developer/reference/backend/security.html`

Same fallback rule as Task 2 if a URL has moved: WebSearch for the official `odoo.com/documentation/17.0/...` replacement, don't substitute a third-party primary source.

- [ ] **Step 2: Append Sections 3–5**

Append to `skills/odoo-dev/odoo-guidance.md`:

```markdown
## 3. Views & QWeb

- **View types** most used in custom modules: `form`, list (XML tag `<tree>` in Odoo 17 — confirm
  against the fetched `views.html` whether this has changed, since the tag name has shifted across
  recent Odoo releases and this section must state whatever's actually current for 17), `kanban`,
  `search`. Each is XML registered via an `ir.ui.view` record, referenced by an
  `ir.actions.act_window`, which is what a menu item opens.
- **View inheritance** (`inherit_id` + `<xpath expr="..." position="...">`) is the standard way
  to modify a view without copying it — `position="after"/"before"/"inside"/"replace"/"attributes"`.
  Prefer targeting a stable `name`/`field name=` attribute in the `expr`, not positional XPath
  (`//field[3]`), since upstream view edits reorder elements and silently break positional xpath.
- **QWeb** (`<t t-...>` templates) is used for both web frontend rendering and PDF reports
  (`report.xml` declares an `ir.actions.report` pointing at a QWeb template `ir.ui.view` of type
  `qweb`). Report templates commonly extend `web.external_layout` for letterhead/footer.
- Domains in `search`/`filter`/field `domain=` attributes use Polish-prefix tuples/strings, e.g.
  `[('state', '=', 'done'), '|', ('user_id', '=', uid), ('team_id', 'in', team_ids)]` — `&`/`|`
  apply to the *next two* leaves, not to everything after them.

**Version notes:** Odoo 17 (current).

## 4. Business logic patterns

- **Wizards** are transient models (`models.TransientModel`) — rows auto-expire (default ~1 day,
  `_transient_max_hours`), so they're for a single UI interaction's temporary state, never for
  data meant to persist.
- **Mixins** (e.g. `mail.thread`, `mail.activity.mixin`) are added via multiple inheritance
  (`_inherit = ['base.model', 'mail.thread']`) to add cross-cutting behavior (chatter, activities)
  without duplicating code — prefer composing an existing mixin over reimplementing its behavior.
- **Scheduled actions** (`ir.cron`) run server-wide, not per-user — code inside must not assume
  `self.env.user` is a real logged-in user with the expected access; explicit `sudo()` or a
  dedicated technical user is typical. Cron jobs share the `max_cron_threads` worker pool (see
  Section 7) — a long-running cron can starve other scheduled jobs, not just itself.
- **Server actions** (`ir.actions.server`) can run Python, but only within what Odoo's sandboxed
  eval allows unless the type is `code` under `base_automation` module trust settings.

**Version notes:** Odoo 17 (current).

## 5. Security

- **`ir.model.access.csv`** grants CRUD *per model, per group* — `perm_read/write/create/unlink`
  as 1/0 columns. A group with no matching row for a model has zero access, silently (no error,
  the record/menu is just invisible).
- **Record rules** (`ir.rule`) filter *which rows* a group can see/act on within a model it
  already has model-level access to, via a domain evaluated with `user` in scope (e.g.
  `[('company_id', '=', user.company_id.id)]`). Global rules (no group) apply to everyone
  including the model's own admin group unless explicitly scoped.
- **`groups=` on a field** (in the Python field definition or a view's `groups=` attribute)
  hides that field from users outside the listed group — this is UI-level; it does not replace a
  record rule or model access, and `sudo()` bypasses it entirely for whoever calls the code.
- Multi-company: `company_id` fields plus record rules are how row-level company isolation is
  enforced — a model intended to be company-scoped needs both the field and a rule, not just one.

**Version notes:** Odoo 17 (current).
```

- [ ] **Step 3: Verify**

Run: `grep -E "^## [0-9]" skills/odoo-dev/odoo-guidance.md`
Expected: five lines, `## 1.` through `## 5.`, in order.

Run: `grep -c "Version notes:" skills/odoo-dev/odoo-guidance.md`
Expected: `5`.

- [ ] **Step 4: Commit**

```bash
git add skills/odoo-dev/odoo-guidance.md
git commit -m "Add odoo-guidance.md sections 3-5 (views/QWeb, business logic, security)"
```

---

### Task 4: Section 6 (ORM performance) — anchored on TPT's measured catalog

**Files:**
- Modify: `skills/odoo-dev/odoo-guidance.md` (append)

**Interfaces:**
- Consumes: existing file from Task 3 — append after Section 5.
- Produces: Section 6. This is the deepest section and the one `findings.md` entries (Task 7+)
  cross-reference by tier name (e.g. "Tier 1: create()/write() overrides assuming a single
  record").

- [ ] **Step 1: No research needed for the core content**

The user supplied a real, measured, tiered catalog of ORM anti-patterns from actual TPT modules
(POS, RMA, stock, sales) — this is the primary content, transcribed faithfully below. Do not
replace or water down these specifics; they're more valuable than generic doc-derived advice.
Optionally supplement with a one-line pointer back to Section 2 for any ORM concept the catalog
assumes (e.g. prefetch sets) — don't re-explain concepts already covered in Section 2.

- [ ] **Step 2: Append Section 6**

Append to `skills/odoo-dev/odoo-guidance.md`:

```markdown
## 6. ORM performance

Real, measured anti-patterns found across TPT modules (POS, RMA, stock, sales), ordered by what
they actually cost. Tier 1 is the category worth grepping for first in any slow flow.

### Tier 1 — tens of seconds each

**O(n²) algorithms.** A lookup per record over a collection that also grows per record. Seen as
a supplier-info lookup inside a loop creating stock moves, and a `write()` override that wrote
the whole recordset once *per record* in it.
→ If a loop contains a `search()` or a `write()` over something that scales with the same input,
it's quadratic. Find those first.

**`create()`/`write()` overrides that assume a single record.** The single biggest category
found — a dozen modules across POS, RMA, stock, and sales each silently turned a batch operation
into a loop.
→ Use `@api.model_create_multi` for create, and inside an override iterate with
`for record in self:` while batching the *queries* the loop issues — not one `write()` per
record.

**Rebuilding an aggregate from scratch on every change.** Changing one order line rebuilt every
summary row of the whole order; deleting a document did the same for its aggregate.
→ Refresh what changed, not everything that could have changed.

**Missing indexes on foreign keys that are checked on delete.** Without an index, Postgres
sequential-scans the referencing table once per deleted row.
→ Any `Many2one` pointing at a document people actually delete needs `index=True`.

**Writing distinct values one record at a time.** The ORM batches pending writes by identical
value, so N genuinely different values means N round trips by construction — there's no batching
to be had from the ORM alone here.
→ When the values are distinct by nature, derive the whole set in one SQL statement instead of
looping `record.write()`.

### Tier 2 — seconds each

**`@api.constrains` used to write instead of to check.** A constraint only fires when its listed
field is part of the `create`/`write` call — a value set by a default, by `_write`, by an
import, or by a migration silently never gets the derived value, and nothing in the data marks
which rows those are. Declared on a stored compute's field, it additionally re-runs on every
recomputation. TPT found 137 methods writing from a constraint, 128 of them inside a loop; one
duplicate-order flow spent 74% of its time in constraint methods.
→ A constraint checks and raises. A stored `compute` with `@api.depends` derives.

**Assigning fields per record instead of per batch.** `record.field = value` on a stored record
is a full `write()` — every override and constraint on that model runs behind it.
→ Collect the values, group by identical value, issue one `write()` per group.

**`search()` per record.** The classic N+1 — one list view ran two searches per row.
→ One `search()` with `('field', 'in', self.ids)`, then distribute the result through a dict
keyed by that field.

**Writing values that didn't change.** Ranking flags, preference flags, aggregate rebuilds — all
rewrote identical values, and every write drags its constraints and computes behind it regardless
of whether the value actually moved.
→ Diff first, write only what moved.

**Writing on read paths.** Computes that write while you read, found across every read-only
screen that used them.
→ Assign inside a compute, never `write()`. Use `read()` for read paths instead of calling a
`_compute_*` method directly (see Tier 3 on why calling it directly is also a correctness bug).

### Tier 3 — small individually, but everywhere in the code

**`exists()` inside a loop.** `exists()` is never prefetched and always issues its own query. One
confirmation flow spent 456 statements re-establishing that records it had just loaded still
existed.
→ `for line in self.exists():` once, outside the loop — not `if line.exists():` inside it.

**Prefetch killers per record.** `sudo()`, `with_context()`, and `browse(single_id)` each
produce a prefetch set of exactly one record, so the *next* field read on it fetches a single row
with all its columns instead of batching with its siblings.
→ One `sudo()` (or `with_context()`) call on the whole recordset up front, not one per record
inside a loop.

**`env.ref()` inside a loop.** It ends in an `exists()` call, so it's one query per call even
though the XML-ID lookup itself is cached.
→ `_xmlid_to_res_model_res_id()` plus `browse()`, resolved once outside the loop.

**Calling compute methods directly.** Calling a `_compute_*` method yourself means the field
isn't protected the way the framework protects it during a real recompute, so every assignment
inside it becomes a real `write()` of the record — with the dependency searches and property
reads that come with a write, not just the field set.
→ `invalidate_recordset([...])` + `modified([...])`, and let the framework re-run the compute
normally.

**Loading full records when only ids are needed.** `browse()` plus a field read fetches every
column the recordset's fields prefetch, even when only the id or one relation column is used.
→ For pure id relationships, read the columns directly (e.g. via `read()` limited to the needed
field, or `_read_group`) instead of browsing full records.

**Copying data that shouldn't follow a duplicate.** Audit trails and change histories copied
along with the document they logged, on every `copy()` — cost during the duplicate, and wrong
data afterwards (a fresh document inheriting another document's history).
→ `copy=False` on any field that records what happened to the *original*, not the copy.

**Version notes:** Odoo 17 (current).
```

- [ ] **Step 3: Verify**

Run: `grep -E "^## [0-9]" skills/odoo-dev/odoo-guidance.md`
Expected: six lines, `## 1.` through `## 6.`, in order.

Run: `grep -c "^### Tier" skills/odoo-dev/odoo-guidance.md`
Expected: `3`.

- [ ] **Step 4: Commit**

```bash
git add skills/odoo-dev/odoo-guidance.md
git commit -m "Add odoo-guidance.md section 6: ORM performance catalog

Transcribes TPT's own tiered catalog of measured ORM anti-patterns
(POS, RMA, stock, sales modules) as the primary content for this
section, per the design spec's exception for section 6."
```

---

### Task 5: Section 7 (Server architecture & resource utilization)

**Files:**
- Modify: `skills/odoo-dev/odoo-guidance.md` (append)

**Interfaces:**
- Consumes: existing file from Task 4 — append after Section 6.
- Produces: Section 7. Must link out to `tpt-dev-scripts`/`tpt-odoo-ops` for TPT's actual
  numbers rather than stating specific per-client values (Global Constraints).

- [ ] **Step 1: Research**

WebFetch:
- `https://www.odoo.com/documentation/17.0/administration/on_premise/deploy.html` (worker model,
  memory limits, `db_maxconn`, `--workers`, `--max-cron-threads`, gevent worker for longpolling —
  this page is the canonical source for all of Section 7's specifics)

Cross-check any numeric formula (e.g. the workers-per-CPU sizing guidance) against this page's
current wording rather than relying on memory of older Odoo versions' docs, since these
recommendations have shifted across releases.

- [ ] **Step 2: Append Section 7**

Append to `skills/odoo-dev/odoo-guidance.md` (verify and adjust any specific numbers below
against the fetched deploy.html page before finalizing — the structure and factors are stable,
the exact formula constants are the part to confirm):

```markdown
## 7. Server architecture & resource utilization

Framework-level only: this section covers how Odoo *itself* consumes and allocates server
resources. TPT's actual per-client worker counts, memory tiers, and ports are **not** repeated
here — see `tpt-dev-scripts`'s `odoo-conf-summary.md` and its `CLAUDE.md` for those, and the
`tpt-odoo-ops` skill for the deploy procedures that apply them.

- **Worker models.** `workers = 0` (the default) runs Odoo single-process, single-threaded,
  synchronous — fine for local dev, wrong for production (one slow request blocks every other
  request on that process). `workers > 0` switches to a **prefork multi-process** model: one
  master process plus N worker processes, each handling one request at a time, plus dedicated
  cron workers (`max_cron_threads`) separate from the HTTP worker pool.
- **Sizing workers to CPU/RAM.** The standard guidance is workers ≈ `(#CPU cores × 2) + 1`, with
  each worker also needing to fit inside available RAM per Odoo's own per-worker memory-limit
  parameters below — CPU count alone isn't sufficient if memory is the tighter constraint on a
  given host. `max_cron_threads` is a *separate* pool on top of `workers` — increasing it doesn't
  reduce HTTP capacity, but it does add to total memory demand.
- **Per-worker memory limits** (`limit_memory_soft` / `limit_memory_hard`): a worker exceeding
  the soft limit is killed *after finishing its current request*, then respawned; exceeding the
  hard limit kills it immediately, mid-request. These exist to contain a single runaway
  request/worker, not to cap the server's total memory — total memory is roughly
  `workers × per-worker footprint`, so raising worker count without checking available RAM
  against these limits is how a host gets OOM-killed under load rather than gracefully degraded.
- **`limit_time_cpu` / `limit_time_real`**: per-request wall-clock/CPU ceilings; a request
  exceeding them is killed. `limit_time_real_cron` is the equivalent for cron jobs, typically set
  higher since batch jobs legitimately run longer than interactive requests.
- **gevent worker (longpolling).** The `--workers`-based prefork pool handles normal HTTP;
  real-time features (chat notifications, `bus.bus` longpolling) need a separate, single
  gevent-based worker (`--gevent-port`) since long-held connections would otherwise tie up a
  regular worker process for the connection's whole lifetime. This is one additional process, not
  part of the `workers` count.
- **DB connection pool (`db_maxconn`)** caps how many Postgres connections *this Odoo instance*
  will open. Each active worker holds at least one connection while handling a request; if
  `db_maxconn` is smaller than concurrent active workers, requests queue for a connection even
  though the workers themselves are idle-waiting, not CPU-bound — a symptom that looks like "not
  enough workers" but is actually "not enough DB connections for the workers you have."
  `db_maxconn` also has to fit within Postgres's own `max_connections` shared across every
  service on that DB host, not just this one Odoo instance.
- **Cron/queue_job concurrency.** Scheduled actions share `max_cron_threads` among themselves —
  a stuck or very long cron job can starve every other scheduled job on that instance, not just
  delay itself. If a project uses the community `queue_job` module for async job processing, its
  channel/concurrency config is a *separate* pool again, layered on top of both `workers` and
  `max_cron_threads` — sizing one without the others is a common way to under- or over-provision.

**Version notes:** Odoo 17 (current).
```

- [ ] **Step 3: Verify**

Run: `grep -E "^## [0-9]" skills/odoo-dev/odoo-guidance.md`
Expected: seven lines, `## 1.` through `## 7.`, in order.

Run: `grep -c "tpt-dev-scripts\|tpt-odoo-ops" skills/odoo-dev/odoo-guidance.md`
Expected: at least `1` (the cross-reference at the top of Section 7).

Run: `grep -E "limit_memory|db_maxconn|workers|gevent" skills/odoo-dev/odoo-guidance.md | wc -l`
Expected: non-zero — confirms the specific framework parameters are present, not just prose.

- [ ] **Step 4: Commit**

```bash
git add skills/odoo-dev/odoo-guidance.md
git commit -m "Add odoo-guidance.md section 7: server architecture & resource utilization"
```

---

### Task 6: Section 8 (Debugging & profiling)

**Files:**
- Modify: `skills/odoo-dev/odoo-guidance.md` (append)

**Interfaces:**
- Consumes: existing file from Task 5 — append after Section 7. This is the last section of the
  file for this plan.
- Produces: Section 8, completing `odoo-guidance.md`.

- [ ] **Step 1: Research**

WebFetch:
- `https://www.odoo.com/documentation/17.0/developer/reference/backend/testing.html` (has some
  profiling/logging references)
- If a dedicated "performance"/"profiling" page exists under
  `https://www.odoo.com/documentation/17.0/developer/` at fetch time, use it as primary source
  for the profiler/`--log-sql` specifics below; otherwise these are stable enough across 17.x
  point releases to write from the outline given.

- [ ] **Step 2: Append Section 8**

Append to `skills/odoo-dev/odoo-guidance.md`:

```markdown
## 8. Debugging & profiling

What makes Sections 6–7 actionable rather than theoretical — how to actually find which of those
patterns is happening, and where.

- **`--log-sql`** (or `log_level = debug_sql` in the config file) logs every SQL statement Odoo
  issues. Combined with a request, this turns "the page is slow" into a concrete, greppable
  statement count and duration — the number Tier 1–3 anti-patterns above show up as directly
  (e.g. "456 statements" for the `exists()`-in-a-loop case).
- **`odoo shell -c <config> -d <db>`** drops into a Python REPL with `env` already bound to the
  target database — the fastest way to reproduce a suspected N+1 or quadratic pattern in
  isolation (time a `search`/`write` against a real recordset) without going through the web UI.
- **Odoo's built-in profiler** (`Settings > Technical > Performance` in developer mode, or the
  `odoo.tools.profiler.Profiler` context manager in code) records per-call SQL and execution-time
  breakdowns and can export a speedscope-compatible trace — useful for confirming *which* method
  in a call stack is the expensive one before assuming.
- **`py-spy`** (`py-spy dump --pid <worker-pid>` or `py-spy record ... --pid <worker-pid>`)
  attaches to a live worker process without restarting it — the right tool when a worker is
  visibly stuck/slow *right now* and restarting would lose the chance to diagnose it.
- **`pg_stat_statements`** (Postgres extension; check it's enabled with
  `SELECT * FROM pg_extension WHERE extname = 'pg_stat_statements';`) tracks aggregate cost
  across *all* queries over time, independent of any one request — the tool for "what's the
  worst query on this database overall," as opposed to `--log-sql`'s per-request view.
- **`EXPLAIN ANALYZE`** on a query pulled from `--log-sql` or `pg_stat_statements` confirms
  whether a suspected missing index (Tier 1) is actually the cause — look for `Seq Scan` on a
  large table where an `Index Scan` would be expected.

**Version notes:** Odoo 17 (current).
```

- [ ] **Step 3: Verify**

Run: `grep -E "^## [0-9]" skills/odoo-dev/odoo-guidance.md`
Expected: eight lines, `## 1.` through `## 8.`, in order — the file is now complete.

Run: `grep -c "Version notes:" skills/odoo-dev/odoo-guidance.md`
Expected: `8`.

- [ ] **Step 4: Commit**

```bash
git add skills/odoo-dev/odoo-guidance.md
git commit -m "Add odoo-guidance.md section 8: debugging & profiling

Completes odoo-guidance.md's initial 8-section structure."
```

---

### Task 7: `findings.md` scaffolding

**Files:**
- Create: `skills/odoo-dev/findings.md`

**Interfaces:**
- Consumes: nothing (independent of Tasks 2–6; could run any time after Task 1).
- Produces: `skills/odoo-dev/findings.md`, referenced by `SKILL.md`'s "Adding a finding" section
  (Task 1) and by the design spec's cross-reference rule for dated, client-attributable
  incidents.

- [ ] **Step 1: Create `skills/odoo-dev/findings.md`**

No fabricated entries — the file starts with only the header and template, per the Global
Constraints. The tiered catalog in `odoo-guidance.md` Section 6 already captures the substance
of what the user supplied; none of it came with a specific date or client name attached, so
nothing gets invented here to force-fit the template. Real dated entries get added later, by
whoever hits them, using this template.

```markdown
# TPT Odoo findings

Real, dated performance/server incidents from TPT client and product work — append-only, never
bulk-regenerated. If a finding is a specific instance of a general pattern already documented in
`odoo-guidance.md`'s ORM performance section, cross-reference that section's tier/pattern name
instead of re-explaining the pattern here.

Only add an entry for something actually observed and measured — no hypothetical or "this could
happen" entries.

## Template

Copy this for each new entry:

```
## <short title> (<client/product>, <date>)
**Symptom:** what was observed
**Root cause:** what was actually wrong
**Fix / takeaway:** what to do differently
```
```

- [ ] **Step 2: Verify**

Run: `test -f skills/odoo-dev/findings.md && grep -c "^## " skills/odoo-dev/findings.md`
Expected: file exists; exactly `1` (only the "## Template" heading — zero real entries).

- [ ] **Step 3: Commit**

```bash
git add skills/odoo-dev/findings.md
git commit -m "Add findings.md skeleton for odoo-dev skill

Starts empty per the design spec -- no fabricated entries. Real
incidents get appended using the template as the team hits them."
```

---

### Task 8: Final consistency pass

**Files:**
- Read-only check across: `skills/odoo-dev/SKILL.md`, `skills/odoo-dev/odoo-guidance.md`,
  `skills/odoo-dev/findings.md`, `skills/README.md`

**Interfaces:**
- Consumes: the finished output of Tasks 1–7.
- Produces: nothing new — this is a verification-only task with no file changes expected. If it
  finds a problem, fix it in place and re-run the check before committing.

- [ ] **Step 1: Check heading structure is complete and in order**

Run: `grep -E "^## [0-9]" skills/odoo-dev/odoo-guidance.md`
Expected: exactly eight lines, `## 1. Module anatomy` through `## 8. Debugging & profiling`, in
that numeric order, no gaps or duplicates.

- [ ] **Step 2: Check every section has its version-notes callout**

Run: `grep -c "Version notes:" skills/odoo-dev/odoo-guidance.md`
Expected: `8` — one per section, none missing.

- [ ] **Step 3: Check cross-references resolve to real paths**

Run (from repo root):
```bash
test -f ../tpt-dev-scripts/odoo-conf-summary.md && echo "odoo-conf-summary.md: OK"
test -f skills/tpt-odoo-ops/SKILL.md && echo "tpt-odoo-ops SKILL.md: OK"
```
Expected: both print `OK`. (If `tpt-dev-scripts` isn't checked out at that relative path in this
environment, confirm the path referenced in prose is still the *name* `tpt-dev-scripts` and
`odoo-conf-summary.md` — the file's exact location may vary by machine, but the name must match
what this repo's own `CLAUDE.md` and `tpt-odoo-ops` already call it.)

- [ ] **Step 4: Check `findings.md` still has zero fabricated entries**

Run: `grep -c "^## " skills/odoo-dev/findings.md`
Expected: `1` (only the template heading) — confirms nothing got invented during the earlier
tasks.

- [ ] **Step 5: Check `skills/README.md`'s index is accurate**

Run: `grep "odoo-dev" skills/README.md`
Expected: one line, matching Task 1's edit, still present and accurate.

- [ ] **Step 6: If any check failed, fix inline and re-run only the failed check.** If all
checks pass, no commit is needed for this task (nothing changed) — plan is complete.
