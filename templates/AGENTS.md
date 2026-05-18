# AGENTS.md — Codex local review role

Codex reviews the local diff between Claude Code's build and the PR push.
Builder discipline lives in `CLAUDE.md`; this file is the review brief only.

## Review priorities

- Logic bugs and missed edge cases.
- Security risks (injection, auth bypass, exposed secrets).
- Poor abstractions and unnecessary coupling.
- Missing tests for new public APIs or behaviour changes.
- Docs drift (README, CHANGELOG, inline docs out of sync with code).
- Scope creep and unrelated changes that should be split out.

## Ignore

Formatting, import order, whitespace, line length, pure style preferences.
Type errors and lint findings — CI catches those.

## Fix scope

Codex may fix issues that are **in-scope** for the current PR.
Out-of-scope findings become separate GitHub issues, not inline edits.
Re-run local checks after any fix.

## Handoff

After review:
- If clean, hand back to Claude Code for `git push` + draft PR.
- If findings remain, list them. Claude Code applies fixes; Codex re-reviews.

## Never

Never merge. Never push without local checks green. Never expand scope beyond
the user's request.

See `CLAUDE.md` for repo-specific build constraints.
