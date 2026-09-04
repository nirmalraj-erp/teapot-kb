# skills/

Canonical copies of Claude Code skills the team has written for TPT work. One directory per
skill (`<skill-name>/SKILL.md`, plus any reference files it needs), matching Claude Code's own
skill layout so a skill here can be dropped straight into a project.

**This is the source of truth — the copy here is the one to edit.** Claude Code only discovers
skills from a project's own `.claude/skills/` or from your user-level `~/.claude/skills/`; it
does not read skills out of an arbitrary sibling repo like `teapot-kb`. To actually use a skill
from here:

- **Available everywhere you work** (recommended for anything not tied to one repo, like
  `tpt-odoo-ops`): copy or symlink the skill directory into your own `~/.claude/skills/`.
  ```bash
  ln -s ~/PycharmProjects/teapot-kb/skills/tpt-odoo-ops ~/.claude/skills/tpt-odoo-ops
  ```
- **Available only in one project** (for a skill tied to that codebase's conventions): copy or
  symlink it into that repo's `.claude/skills/` instead.

If you improve a skill from either location, port the change back here so the canonical copy
stays current — don't let copies drift the way `tpt-odoo-ops` did before this folder existed
(it was hand-duplicated, byte-identical, in both `tpt-dev-scripts` and `odoo-17/tpt-eu`).

Current skills: `tpt-odoo-ops` (TPT Odoo production deploy/ops runbook).
