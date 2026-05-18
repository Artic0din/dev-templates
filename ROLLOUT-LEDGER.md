# Discipline Stack Rollout Ledger

One-line-per-repo ledger of scaffold rollout state. Append a new row per scaffold attempt.

Columns:
- **Repo** — owner/repo
- **Baseline** — dev-templates tag at scaffold time
- **PR** — scaffold PR url
- **CI** — `green` / `red` / `partial` (stack jobs green, repo jobs red)
- **Stack issues** — bugs found in dev-templates during this scaffold
- **Repo issues** — pre-existing repo-side problems exposed by the scaffold
- **Next action** — what unblocks the next step

---

## Pilots

| Repo | Baseline | PR | CI | Stack issues | Repo issues | Next action |
|---|---|---|---|---|---|---|
| `Artic0din/powerbot` | v1.1.0 | [#11](https://github.com/Artic0din/powerbot/pull/11) | partial | claude-review-gate startup_failure (Action optional in v1.0.5, still unresolved) | eslint missing in `bot/` subpackage, typecheck setup, gitleaks finding, dependency-review fail, version drift in monorepo | Address `bot/` package eslint + typecheck setup; uninstall CodeRabbit + Sourcery GitHub Apps |
| `Artic0din/ha-pricehawk` | v1.1.4 | [#83](https://github.com/Artic0din/ha-pricehawk/pull/83) | partial | v1.1.1 branch-filter, v1.1.2 changelog regex, v1.1.3 set-e short-circuit, v1.1.4 pipefail-on-empty-grep — all fixed; baseline now v1.1.5 | No `pyproject.toml` (requires uv migration), 5 repo-specific workflows overlap ci-core, AGENTS.md content convention | Separate PR: uv migration. Separate PR: workflow consolidation |

## Phase 4

| Repo | Baseline | PR | CI | Stack issues | Repo issues | Next action |
|---|---|---|---|---|---|---|
| `Artic0din/GridWise` | — | [#176 CLOSED](https://github.com/Artic0din/GridWise/pull/176) | — | — | No longer actively developed; clean-slate stub since 2026-05-04. Bootstrap PR closed without merge. | **Dropped from rollout.** Reopen #176 if GridWise development resumes. |
| `Artic0din/PowerSync` (bootstrap, fork-only) | v1.1.5 (policy ref) | [#233](https://github.com/Artic0din/PowerSync/pull/233) | pending | None — bootstrap is uv init only | HACS integration with no Python tooling. Bootstrap PR adds `pyproject.toml` (deps mirror manifest.json: aiohttp, aemo-to-tariff, cryptography, goodwe, protobuf, scipy), `uv.lock` (32 packages), `CHANGELOG.md` (fork-only). 487 tests collect locally. | Merge #233 to fork's `main`, THEN open fork-only scaffold PR. Base = `Artic0din/PowerSync:main`, never upstream. |
| `Artic0din/PowerSync` (scaffold, fork-only) | v1.1.5 | [#234](https://github.com/Artic0din/PowerSync/pull/234) | partial | Lint job lacks `pull-requests: write` permission → reviewdog can't post inline annotations, dumps rdjson to stdout + exits 1. Functionality intact (correctly fails on real findings), UX degraded. Optional v1.1.6 fix. | Repo lint findings (unsorted imports, unused imports, `datetime.UTC` upgrades) — real ruff issues, fix in separate PR. Pyright can't resolve `homeassistant.*` — needs `homeassistant` in pyproject dev group. Dependency-review: moderate-severity advisory in a dep (bump needed). | Stack validation PASS: 487 tests pass, all 4 managed workflows fire, version-drift correct, CR cleanup ran, AGENTS.md created. Repo follow-up: lint + pyright + deps. |

## Excluded from rollout

- **`Artic0din/GridWise`** — no longer actively developed (clean-slate stub).
- **Oversight** (Power BI) — no CI pattern fits.
- **Watchbill** (HTML + Excel/VBA) — no CI pattern fits.

---

## Stack version history

| Tag | Date | Change |
|---|---|---|
| v1.0.0 | 2026-05-18 | Initial release — full ci-core, claude-review-gate with `always() + needs:` |
| v1.0.1 | 2026-05-18 | `pnpm test` fallback when no `test:coverage` script |
| v1.0.2 | 2026-05-18 | Secrets `required: false` so callers without secrets don't startup_fail |
| v1.0.3 | 2026-05-18 | `anthropics/claude-code-action@v0` (no v1 exists); `mode: experimental-review`; `override_prompt` from prompt file |
| v1.0.4 | 2026-05-18 | `!cancelled()` replaces `always()` on gate (attempted fix, didn't resolve) |
| v1.0.5 | 2026-05-18 | claude-review + gate disabled in ci-core (startup_failure root cause unknown) |
| v1.1.0 | 2026-05-18 | Three-agent migration: `*.instructions.md` rename, AGENTS.md template, 4 separate managed workflows, scaffold v1.0.x → v1.1.0 migration logic |
| v1.1.1 | 2026-05-18 | security-scan + docs-check trigger on all PRs (not just `main`-base) |
| v1.1.2 | 2026-05-18 | version-drift regex skips non-semver headings |
| v1.1.3 | 2026-05-18 | version-drift uses `if/then/fi` instead of `[[ ]] && cmd` (set -e safety) |
| v1.1.4 | 2026-05-18 | version-drift tolerates empty `find \| grep` pipeline (pipefail) |
| **v1.1.5** | **2026-05-18** | **Locked baseline.** Python prereq warning, AGENTS.md append behavior, Phase 4 order documented |

---

## Operating rules (from locked v1.1.5 policy)

1. **No architecture changes** beyond v1.1.5 unless a new repo exposes a real stack bug.
2. **Python repos must have `pyproject.toml` + uv** before scaffold runs. Migrate in a separate PR.
3. **Scaffold PRs prove wiring**, not repo health. Repo-side fixes are separate PRs.
4. **AGENTS.md append-only.** Never rename existing AGENTS.md to PROJECT.md.
5. **Phase 4 order**: GridWise → PowerSync → PLNR → Arsenal → KiloWasps → Amprage → SpoolWise.
6. **Oversight + Watchbill excluded** until separate patterns defined.
7. **Don't start the next repo** until the previous repo's scaffold PR result is known.
