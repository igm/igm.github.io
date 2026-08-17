# CLAUDE.md

Instructions for Claude Code in this repo.

Read these two, in order — they are the source of truth and are not duplicated here:

1. **[README.md](README.md)** — what the site is, how to add a post, how unlisted
   posts work, the repo layout, the design system, deployment, and previewing.
2. **[AGENTS.md](AGENTS.md)** — the agent-facing invariants: no build step, no
   third-party requests, the tiny JS budget, and the traps around intentionally
   unlisted posts.

If guidance here would ever conflict with the README, the README wins — update
it there rather than adding a second copy of the rules.

## Quick reminders

- Plain HTML, no generator: every page is served exactly as committed, and each
  one carries the full shell inline. Shell changes touch every page.
- Verify in a browser before reporting done — light and dark themes, plus a
  narrow viewport. `python3 -m http.server` in the repo root, or open the file
  directly; both must work.
- Do not commit or push unless asked.
