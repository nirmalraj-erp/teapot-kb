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
