# runbooks/

Step-by-step operational procedures — the "what to do when X" docs, as opposed to the static
reference info in `servers/`. One file per procedure: `<slug>.md`.

Examples: onboarding a new client server, deploying a config change safely, restarting/
troubleshooting a stuck Odoo instance, renewing a cert, recovering from a port collision.

Write these as procedures a team member can follow under pressure — concrete commands and
checks, not prose explanations of the system (that belongs in `servers/` or the relevant
repo's `CLAUDE.md`).
