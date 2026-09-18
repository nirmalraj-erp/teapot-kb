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
