---
name: save
description: Write a _journal/YYYY-MM-DD.md entry for this session, then git add ., commit, and push. Use when the user says "save", "/save", or wants to wrap up a session in ~/Projects/dcs.spa.
---

# save

Wrap up the current session for the DCS project.

## Steps

1. **Check state** — Run `git status` and `git log --oneline -5` in parallel so you know what changed and can match the repo's commit-message style.

2. **Journal entry** — Write to `_journal/<today>.md` (today = `date +%Y-%m-%d`):
   - If the file does not exist, create it with an H1 of the date.
   - If it already exists, append a new H2 section (timestamp or short heading) — never overwrite.
   - Summarise what actually changed in this session and *why*. Bullet points. Terse. Reference file paths where useful. Skip boilerplate.
   - Create `_journal/` if missing.

3. **Stage** — `git add .` (repo-wide; the working tree should already be clean of anything unrelated at this point — if `git status` shows stray files, flag them to the user before staging).

4. **Commit** — Single commit covering the session, unless the user asked for multiple. Use a HEREDOC message matching the repo's imperative short-title style (e.g. "Fix carousel mode switch: prep state before toggling fade-mode class"). Include the standard `Co-Authored-By` trailer per the user's global commit policy.

5. **Push** — `git push`. Note to the user that GitHub Pages typically rebuilds in 30–90s.

## Rules

- Never use `--no-verify` or other hook-bypass flags.
- Never force-push.
- If `.env`, `*.key`, credentials, or other secrets appear in `git status`, stop and warn the user instead of staging.
- If the working tree is clean, don't create an empty commit — just tell the user there's nothing to save.
- Don't narrate each step verbosely — run the commands, report the commit SHA and push result.
