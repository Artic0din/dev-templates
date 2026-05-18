# CLAUDE.md — Claude Code builder discipline

Claude Code is the builder. Codex (see `AGENTS.md`) reviews locally before PR.
GitHub Copilot reviews inline on PR. CI verifies. Ryan merges.

## Build discipline

- Implement only what the user requested. No scope creep, no speculative features.
- Tests in the same PR for new public APIs, endpoints, DB ops, or behaviour changes.
- Update `CHANGELOG.md` under `[Unreleased]` every PR.
- Update inline docs / docstrings when signatures change.
- Keep PRs small and focused. One logical change per PR.

## Local checks (every push)

Run the language-appropriate set before push:

- TypeScript: `pnpm lint && pnpm exec tsc --noEmit && pnpm test:coverage && pnpm build`
- Python: `uv run ruff check && uv run pyright && uv run pytest`
- Swift: `swiftlint --strict && swift build && swift test`

Never push with failing checks.

## Secrets

- No `.env`, `.pem`, `.key`, `credentials.json`, tokens, or API keys in git.
- Run `git diff --staged | grep -iE "key|secret|token|password"` before push.

## Conventional commits

`{type}({scope}): {description}` — types: feat, fix, test, refactor, perf, docs, style, chore, ci, build, revert.

## Handoff to Codex

Once local checks pass:
1. Stage the diff.
2. Hand off to Codex for local review (see `AGENTS.md`).
3. Apply any Codex fixes; re-run checks.
4. Open a **draft** PR. Never auto-merge.

## Never

Never auto-merge. Never push with failing checks. Never expand scope beyond
the explicit request.
