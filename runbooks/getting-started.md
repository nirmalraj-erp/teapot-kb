# Getting started with teapot-kb

For anyone new to this KB — dev or functional consultant — on a machine that doesn't have it
yet.

## 1. Get access

`teapot-kb` is a private repo on GitHub: `github.com:nirmalraj-erp/teapot-kb`. You need to be
added as a collaborator (or org member with access) before cloning — ask whoever administers
the GitHub org. You'll also need an SSH key registered on your GitHub account if cloning over
SSH (the remote here uses `git@github.com:...`, not HTTPS).

## 2. Install Claude Code

This KB is designed to be read by Claude Code, not just browsed as plain markdown — its
`CLAUDE.md` is auto-loaded as project context the moment you open a session inside the repo.
See https://docs.claude.com/en/docs/claude-code for install instructions for your platform.

## 3. Clone and open

```bash
git clone git@github.com:nirmalraj-erp/teapot-kb.git
cd teapot-kb
claude
```

That's it — no build step, no dependencies. `CLAUDE.md` loads automatically and gives Claude
the full map of TPT's products, clients, servers, and how this repo is organized.

**You do not need to clone the other four repos** (`odoo-17`, `tpt-dev-scripts`, `viswox`,
`viswox-static-website`) just to use this KB. `CLAUDE.md` and the files under `clients/`/
`products/` reference paths like `~/PycharmProjects/odoo-17/tpt-ind` because that's where they
live on developer machines — those paths won't exist on yours unless you also clone that repo,
and that's expected, not an error. Functional consultants in particular will mostly work
entirely within this repo.

## 4. What you can actually do here

Some examples of things to ask Claude once you're in a `teapot-kb` session:

- "What do we know about `<client>`?" — reads `clients/<domain>/<client>.md` if it exists.
- "Which host is `<client>` deployed on, and what's its status?" — `servers/`,
  `servers/client-repo-matrix.md`.
- "What is VisWoX / XPOS Tech / XMart?" — `products/`.
- "Add a note to `<client>`'s file: <contact info / decision / history>" — Claude will edit the
  right file directly; check each folder's `README.md` first if unsure where something belongs.
- "How do I onboard a new client server?" / "How do I check for duplicate certbot timers?" —
  `runbooks/`.

## 5. Contributing back

Each top-level folder's `README.md` is the authority on what belongs in it and how files there
are named — check it before adding something new. Commit with a clear message describing what
changed and why; push to `main` (no branch/PR process yet for this repo).
