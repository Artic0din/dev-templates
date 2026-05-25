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
| `Artic0din/PowerSync` (scaffold, fork-only) | v1.1.5 (effective) | [#234](https://github.com/Artic0din/PowerSync/pull/234) | partial | Lint job lacks `pull-requests: write` permission → reviewdog can't post inline annotations, dumps rdjson to stdout + exits 1. Functionality intact (correctly fails on real findings), UX degraded. Optional v1.1.6 fix. | Repo lint findings (unsorted imports, unused imports, `datetime.UTC` upgrades) — real ruff issues, fix in separate PR. Pyright can't resolve `homeassistant.*` — needs `homeassistant` in pyproject dev group. Dependency-review: moderate-severity advisory in a dep (bump needed). | Stack validation PASS: 487 tests pass (cached run pre-v1.2.x). **NOTE:** v1 floating now points to v1.2.3 which lacks Python path. Next retrigger of PowerSync CI will all-skip on `language: python`. v1.3.0 restores Python. |
| `Artic0din/PLNR` (scaffold, monorepo+npm) | v1.2.3 | [#141 CLOSED](https://github.com/Artic0din/PLNR/pull/141) | **PARKED — closed** | Multi-path ci-core issue (Python+TS-pnpm+TS-npm+Swift+codecov in one file) triggers GH Actions startup_failure on calls with certain caller inputs. Confirmed real bug: codecov-action@v4 with empty CODECOV_TOKEN startup_failures. Rapid v1 force-pushes during debug also produced non-deterministic resolution. | Repo issues unknown — never reached real CI execution. | **Closed.** Superseded by PR #142. |
| `Artic0din/PLNR` (re-do, v1.3.0-rc.1) | v1.3.0-rc.1 | [#142](https://github.com/Artic0din/PLNR/pull/142) | partial | None — workflow started, all 4 jobs ran. | Real lint/typecheck/tests/gitleaks failures in repo code. | Stack validation **PASS** for TS+npm+monorepo path. Repo follow-ups separate. |
| `Artic0din/Amprage` (scaffold, swift) | v1.3.0-rc.1 | [#1](https://github.com/Artic0din/Amprage/pull/1) | partial | None — workflow started on macos-latest, all 3 jobs ran. `swift test` failed (Amprage uses Xcode project, no `Package.swift`) — real failure, NOT startup_failure. | Test job: `swift test` expects SwiftPM. Lint job: swiftformat/swiftlint findings. Gitleaks: expected on a years-old repo. | Stack validation **PASS** for Swift path. Xcode-project test support is a v1.4 enhancement. |

## v2 baseline

| Tag | Date | Notes |
|---|---|---|
| `v2.0.0` | 2026-05-25 | **Copier scaffolding base** (PR #6). `copier.yml` + `template-ha-component` skeleton shipped. Per-language `ci-core-*.yml@v2` carried forward unchanged from v1.3.0. `scripts/scaffold-discipline` rewritten as thin Copier wrapper (~63 lines, PR #7). Floating `v2` tag tracks latest v2.x.y per `pinning policy`. |

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
| v1.1.5 | 2026-05-18 | Locked baseline. Python prereq warning, AGENTS.md append behavior, Phase 4 order documented |
| v1.1.6 | 2026-05-18 | Lint job permissions. Added `pull-requests: write` so reviewdog can post inline annotations. Discovered via PowerSync #234 |
| v1.2.0 | 2026-05-18 | Monorepo + npm support. Added `working_directory` (default `.`) and `package_manager` (default `pnpm`, accepts `npm`) inputs to ci-core. Backward-compatible. Discovered via PLNR Phase 4 inspection — all 3 active TS app repos use npm; PLNR is monorepo with app in `nesc-scheduler/`. |
| v1.2.1 | 2026-05-18 | working-directory moved to per-step (not `defaults.run.working-directory`) for reusable-workflow compatibility. |
| v1.2.2 | 2026-05-18 | Restored full ci-core after cache clears. |
| ⚠ v1.2.3 (transition) | 2026-05-18 | TS+npm + codecov env-gate only. Python + pnpm + Swift paths dropped. **DO NOT USE as baseline.** Transitional state during bisection of multi-path startup_failure. Next stable baseline is v1.3.0 (per-language ci-core split). See `BUILDER-BRIEF-v1.3.0.md`. |
| v1.3.0-rc.1 | 2026-05-18 | Per-language ci-core split. `ci-core-python.yml`, `ci-core-typescript.yml`, `ci-core-swift.yml`. Monolithic `ci-core.yml` deleted. Codecov env-gate carried forward. |
| **v1.3.0** | **2026-05-18** | **Final.** All 4 language paths real-caller-validated under `@v1.3.0-rc.1`: ✅ Python (PowerSync #234 → `failure`, jobs ran), ✅ TS+pnpm (PowerBot #11 → `failure`, jobs ran), ✅ TS+npm (PLNR #142 → `failure`, jobs ran), ✅ Swift (Amprage #1 → `failure`, jobs ran). No `startup_failure` on any. Floating `v1` moved. Acceptance criteria #1-#10 met. |

---

## Operating rules (from locked v1.1.5 policy)

1. **No architecture changes** beyond v1.1.5 unless a new repo exposes a real stack bug.
2. **Python repos must have `pyproject.toml` + uv** before scaffold runs. Migrate in a separate PR.
3. **Scaffold PRs prove wiring**, not repo health. Repo-side fixes are separate PRs.
4. **AGENTS.md append-only.** Never rename existing AGENTS.md to PROJECT.md.
5. **Phase 4 order**: GridWise → PowerSync → PLNR → Arsenal → KiloWasps → Amprage → SpoolWise.
6. **Oversight + Watchbill excluded** until separate patterns defined.
7. **Don't start the next repo** until the previous repo's scaffold PR result is known.
