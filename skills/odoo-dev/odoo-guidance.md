# Odoo 17 development guidance

Framework-level reference for writing and troubleshooting Odoo 17 custom modules. Baseline is
Odoo 17; see `SKILL.md`'s "Refreshing this guide" procedure for how later versions get added.
This is deliberately lean outside the performance/server sections (6–7) — enough to write
correct code, not an exhaustive restatement of Odoo's own documentation.

## 1. Module anatomy

A module is a directory containing `__manifest__.py` plus the standard subfolders:

- `__manifest__.py` — name, version (by convention, often `17.0.x.y.z`), `depends`, `data`
  (XML/CSV files loaded at install/update, in listed order), `installable`, `application`.
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
  frontend assets, declared via an `assets` key in the manifest (Odoo 17; Odoo 14 and earlier
  used a `web.assets_backend` XML template instead — not relevant to 17 but worth knowing if
  reading older module code).
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
  modified environment — they don't mutate `self`. `sudo()` bypasses ACLs, record rules, and
  field-level `groups=` restrictions — only hard-coded Python checks (e.g. explicit
  `has_group()` calls) are unaffected by it.
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

## 3. Views & QWeb

- **View types** most used in custom modules: `form`, list (XML tag is still `<tree>` in Odoo
  17; Odoo 18 renamed the root tag to `<list>`, so don't carry that over when reading newer
  docs/modules), `kanban`, `search`. Each is XML registered via an `ir.ui.view` record,
  referenced by an `ir.actions.act_window`, which is what a menu item opens.
- **View inheritance** (`inherit_id` + `<xpath expr="..." position="...">`) is the standard way
  to modify a view without copying it — `position="after"/"before"/"inside"/"replace"/"attributes"`.
  Prefer targeting a stable `name`/`field name=` attribute in the `expr`, not positional XPath
  (`//field[3]`), since upstream view edits reorder elements and silently break positional xpath.
- **QWeb** (`<t t-...>` templates) is used for both web frontend rendering and PDF reports
  (`report.xml` declares an `ir.actions.report` pointing at a QWeb template `ir.ui.view` of type
  `qweb`). Report templates commonly extend `web.external_layout` for letterhead/footer.
- Domains in `search`/`filter`/field `domain=` attributes use prefix-notation tuples/strings,
  e.g. `[('state', '=', 'done'), '|', ('user_id', '=', uid), ('team_id', 'in', team_ids)]` —
  `&`/`|` apply to the *next two* leaves, not to everything after them.

**Version notes:** Odoo 17 (current).

## 4. Business logic patterns

- **Wizards** are transient models (`models.TransientModel`) — rows auto-expire based on
  `_transient_max_hours` (default 1 hour of inactivity), cleaned up by a daily "Base: Auto-vacuum
  internal data" scheduled action — so they're for a single UI interaction's temporary state,
  never for data meant to persist.
- **Mixins** (e.g. `mail.thread`, `mail.activity.mixin`) are added via multiple inheritance
  (`_inherit = ['base.model', 'mail.thread']`) to add cross-cutting behavior (chatter, activities)
  without duplicating code — prefer composing an existing mixin over reimplementing its behavior.
- **Scheduled actions** (`ir.cron`) run server-wide, not per-user — code inside must not assume
  `self.env.user` is a real logged-in user with the expected access; explicit `sudo()` or a
  dedicated technical user is typical. Cron jobs share the `max_cron_threads` worker pool (see
  Section 7) — a long-running cron can starve other scheduled jobs, not just itself.
- **Server actions** (`ir.actions.server`) with `state='code'` execute Python through
  `safe_eval` — a restricted eval that blocks arbitrary imports/dangerous builtins, but is
  explicitly documented as *not* a full security sandbox against a determined trusted user. Only
  admin/technical-settings users can create or edit them, and the code runs with whatever access
  the calling context already has (no automatic `sudo()`).

**Version notes:** Odoo 17 (current).

## 5. Security

- **`ir.model.access.csv`** grants CRUD *per model, per group* — `perm_read/write/create/unlink`
  as 1/0 columns. A group with no matching row for a model has zero access, silently (no error,
  the record/menu is just invisible) — access rights are additive across a user's groups (the
  union of whatever each group grants).
- **Record rules** (`ir.rule`) filter *which rows* a group can see/act on within a model it
  already has model-level access to, via a domain evaluated with `user` in scope (e.g.
  `[('company_id', '=', user.company_id.id)]`). Group-scoped rules *unify* (any matching rule
  grants access); global rules (no group) *intersect* with everything else and apply to everyone,
  including the model's own admin group, unless explicitly scoped otherwise.
- **`groups=` on a field** (in the Python field definition or a view's `groups=` attribute)
  hides that field from users outside the listed group and rejects explicit reads/writes to it
  from those users — this is enforced server-side, not just a UI hint, but it does not replace a
  record rule or model access, and `sudo()` bypasses it entirely for whoever calls the code.
- Multi-company: `company_id` fields plus record rules are how row-level company isolation is
  enforced — a model intended to be company-scoped needs both the field and a rule, not just one.

**Version notes:** Odoo 17 (current).

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

## 7. Server architecture & resource utilization

Framework-level only: this section covers how Odoo *itself* consumes and allocates server
resources. TPT's actual per-client worker counts, memory tiers, and ports are **not** repeated
here — see `tpt-dev-scripts`'s `odoo-conf-summary.md` and its `CLAUDE.md` for those, and the
`tpt-odoo-ops` skill for the deploy procedures that apply them.

- **Worker models.** `workers = 0` (the default) runs Odoo **multi-threaded**: a new thread is
  spawned per incoming HTTP request, all inside one process — fine for local dev, wrong for
  production (one slow or stuck request can starve the whole process). `workers > 0` switches to
  **multi-processing (prefork)**: a pool of worker processes is created at startup, each
  handling one request at a time, plus a separate cron worker pool (`max_cron_threads`,
  decoupled from the HTTP worker pool).
- **Sizing workers to CPU/RAM.** Odoo's own rule of thumb is workers ≈ `(#CPU cores × 2) + 1`,
  with roughly "1 worker ≈ 6 concurrent users" as the corresponding load estimate. That's a CPU-only
  starting point — Odoo's docs pair it with a RAM check: estimate needed RAM as
  `#workers × (light_ratio × light_worker_RAM + heavy_ratio × heavy_worker_RAM)`, using ballpark
  figures of a lightweight request costing ~150MB and a heavy request ~1GB, with a typical
  workload assumed ~80% light / 20% heavy. CPU count alone isn't sufficient if memory is the
  tighter constraint on a given host. `max_cron_threads` is a *separate* pool on top of `workers`
  — increasing it doesn't reduce HTTP capacity, but it does add to total memory demand.
- **Per-worker memory limits** (`limit_memory_soft` / `limit_memory_hard`): these cap how much
  memory a single worker process may consume before Odoo recycles it — a worker over the soft
  limit is recycled once it finishes its current request, while a worker over the hard limit is
  killed immediately, mid-request. They exist to contain a single runaway request/worker, not to
  cap the server's total memory — total memory is still roughly `workers × per-worker footprint`,
  so raising worker count without checking available RAM against these limits (and against the
  RAM-estimation approach above) is how a host gets OOM-killed under load rather than gracefully
  degraded.
- **`limit_time_cpu` / `limit_time_real`**: per-request CPU-time/wall-clock ceilings; a request
  exceeding them is killed. `limit_time_real_cron` is the equivalent for cron jobs, typically set
  higher since batch jobs legitimately run longer than interactive requests.
- **LiveChat worker (websocket).** In multi-processing mode, Odoo automatically starts one
  additional, dedicated worker — the LiveChat worker, gevent-based, listening on `--gevent-port`
  — to handle real-time features (chat notifications, `bus.bus`) over websocket connections
  (paths under `/websocket/`, routed to it by the reverse proxy). This is necessary because a
  long-held websocket connection would otherwise tie up a regular prefork worker for the
  connection's whole lifetime. It is one additional process, separate from and not counted in the
  `workers` total. (Older Odoo versions called this the "longpolling" worker/port; the mechanism
  is now websocket-based but serves the same purpose.)
- **DB connection pool (`db_maxconn`, default 64)** is a **per-process** pool: in multi-processing
  mode, each worker process (and the cron worker pool, and the LiveChat/gevent worker) gets its
  own independent pool of up to `db_maxconn` connections — it is not one shared instance-wide pool
  that all workers draw from and queue against. The real multi-process risk is the *aggregate*
  connection count against Postgres's own `max_connections` (shared across every service on that
  DB host, not just this Odoo instance):
  `(1 + workers + max_cron_threads) × db_maxconn` must stay under Postgres's `max_connections`,
  or new connections start failing outright across the whole instance — the leading `1` here is
  the LiveChat/gevent worker's pool, not the master process (the prefork master supervises the
  worker pool but doesn't itself hold a `db_maxconn` pool). (The "workers queue for a
  connection" failure mode is a real risk too, but it applies to `workers = 0` multi-threaded
  mode, where `db_maxconn` *is* one shared pool for the single process — not to the multi-process
  model the rest of this section describes.)
- **Cron/queue_job concurrency.** Scheduled actions share `max_cron_threads` among themselves —
  a stuck or very long cron job can starve every other scheduled job on that instance, not just
  delay itself. If a project uses the community `queue_job` module for async job processing, its
  channel/concurrency config is a *separate* pool again, layered on top of both `workers` and
  `max_cron_threads` — sizing one without the others is a common way to under- or over-provision.

**Version notes:** Odoo 17 (current).

## 8. Debugging & profiling

What makes Sections 6–7 actionable rather than theoretical — how to actually find which of those
patterns is happening, and where.

- **`--log-sql`** (equivalent to `--log-handler=odoo.sql_db:DEBUG`; or `log_level = debug_sql` in
  the config file) logs every SQL statement Odoo issues, with execution time. Combined with a
  request, this turns "the page is slow" into a concrete, greppable statement count and duration
  — the number Tier 1–3 anti-patterns above show up as directly (e.g. "456 statements" for the
  `exists()`-in-a-loop case).
- **`odoo shell -c <config> -d <db>`** drops into a Python REPL with `env` already bound to the
  target database — the fastest way to reproduce a suspected N+1 or quadratic pattern in
  isolation (time a `search`/`write` against a real recordset) without going through the web UI.
- **Odoo's built-in profiler** (`odoo.tools.profiler.Profiler` context manager in code, or from
  the UI: enable developer mode, then **Settings → General Settings → Performance** to turn
  profiling on for the database with an expiry time, then toggle "Enable profiling" again in the
  developer mode tools menu to turn it on for your own session) records per-call SQL and
  execution-time breakdowns via pluggable collectors (SQL, Periodic/traces, QWeb, and a
  high-overhead Sync collector). Results are saved as `ir.profile` records; opening one launches
  the bundled speedscope viewer in a new tab for a flamegraph view — useful for confirming *which*
  method in a call stack is the expensive one before assuming.
- **`py-spy`** (`py-spy dump --pid <worker-pid>` for an instant stack snapshot, or
  `py-spy record -o profile.svg --pid <worker-pid>` to sample over a period and produce a flame
  graph) attaches to a live worker process without restarting it — the right tool when a worker
  is visibly stuck/slow *right now* and restarting would lose the chance to diagnose it.
- **`pg_stat_statements`** (Postgres extension; check it's enabled with
  `SELECT * FROM pg_extension WHERE extname = 'pg_stat_statements';` — it also needs
  `shared_preload_libraries = 'pg_stat_statements'` set at the Postgres server level, which is an
  infra-side change, not something a project can turn on for itself) tracks aggregate cost across
  *all* queries over time, independent of any one request — the tool for "what's the worst query
  on this database overall," as opposed to `--log-sql`'s per-request view.
- **`EXPLAIN ANALYZE`** on a query pulled from `--log-sql` or `pg_stat_statements` confirms
  whether a suspected missing index (Tier 1) is actually the cause — look for `Seq Scan` on a
  large table where an `Index Scan` would be expected.

**Version notes:** Odoo 17 (current).
