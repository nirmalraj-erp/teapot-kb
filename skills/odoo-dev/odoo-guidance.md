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
