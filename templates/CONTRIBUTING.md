# Contributing

## Branches and PRs

- Feature branches off `main`. No direct commits to `main`.
- Conventional commit messages: `{type}({scope}): {description}`.
- Valid types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`,
  `perf`, `build`, `revert`, `style`.
- PR title follows the same convention (squash merge puts PR title in
  history).

## Draft PRs

Draft PRs skip Claude review; the merge gate passes open until you mark
Ready for review.

## CI gate

Branch protection requires the literal `CI passed` check. Individual job
results (lint, typecheck, tests, codecov, claude-review-gate) feed into
the rollup via `needs:`.

## AI review

Claude Code reviews every non-draft PR with tiered findings:
- **Tier 1** blocks merge (security, logic bugs, missing tests for new
  public APIs).
- **Tier 2** records concerns (perf, API design, docs drift).
- **Tier 3** strong-justification only (architecture, naming).

Override the gate with the `ai-review-override` label. Use only for:
- Anthropic API outage (Claude review didn't run)
- Trailer malformed but actual review fine
- Production incident requiring immediate merge

Document the reason in the PR body when applying the label.

## Local checks before push

Run the same checks CI runs:
- TypeScript: `pnpm lint && pnpm exec tsc --noEmit && pnpm test:coverage && pnpm build`
- Python: `uv run ruff check && uv run pyright && uv run pytest`
- Swift: `swiftlint && swift build && swift test`

Never push with failing local checks.

## Secrets

No `.env`, `.pem`, `.key`, `credentials.json`, tokens, or API keys in git.
Run `git diff --staged | grep -iE "key|secret|token|password"` before push.
