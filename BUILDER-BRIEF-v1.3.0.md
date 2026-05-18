# Builder Brief — v1.3.0 Refactor

**Status:** ready to start (separate focused session).
**Predecessor:** v1.2.3 (transition tag — unstable; do not use as baseline).
**Scope:** architectural refactor only. No Phase 4 rollout work in this session.

---

## Why v1.3.0

The monolithic `ci-core.yml` carrying conditionals for Python + TS (pnpm) +
TS (npm) + Swift in a single file hit GitHub Actions `startup_failure` when
called from PLNR's PR #141. Bisection across 11+ revisions confirmed:

- Each language path works in isolation.
- Combination doesn't (specifically Python steps + TS+npm steps + codecov
  step coexisting in one ci-core.yml triggers startup_failure on calls with
  certain caller inputs).
- Rapid `v1` floating-tag force-pushes during debug also produced
  non-deterministic resolution (same content sometimes worked, sometimes
  startup_failed minutes apart).

The fix is structural, not patch-by-patch.

---

## Scope

Split `ci-core.yml` into three per-language reusable workflows:

```
.github/workflows/
  ci-core-python.yml
  ci-core-typescript.yml
  ci-core-swift.yml
```

Each file is small, has no cross-language conditionals, and is independently
testable.

---

## Per-language designs

### ci-core-python.yml

**Inputs:**
- `working_directory` (default `.`)

**Assumes:** caller repo has `pyproject.toml` + `uv.lock` at `working_directory`.

**Jobs:**
- `lint` — uv sync → ruff check (reviewdog on PRs) → ruff format --check
- `typecheck` — uv sync → pyright
- `tests` — uv sync → pytest --cov --cov-report=xml → codecov upload
  (env-gated, see below)
- `gitleaks` — fetch-depth 0 → gitleaks-action

**Permissions:**
- `lint` — `contents: read, pull-requests: write` (reviewdog inline)

### ci-core-typescript.yml

**Inputs:**
- `working_directory` (default `.`)
- `package_manager` (default `pnpm`, accepts `npm`)

**Assumes:** caller repo has `package.json` + lockfile at `working_directory`.

**Jobs:**
- `lint` — install (pnpm/npm based on input) → lint
- `typecheck` — install → tsc --noEmit
- `tests` — install → test:coverage (with jq fallback to test) → codecov
  upload (env-gated)
- `gitleaks` — same as Python

### ci-core-swift.yml

**Inputs:**
- `working_directory` (default `.`)

**Assumes:** caller repo has `Package.swift` or Xcode project at
`working_directory`. Runner: `macos-latest`.

**Jobs:**
- `lint` — brew install swiftformat swiftlint → swiftformat --lint → swiftlint
- `tests` — swift test --enable-code-coverage
- `gitleaks` — same as Python (ubuntu-latest for this job)

---

## Codecov env-gate (carry forward from v1.2.3)

`codecov-action@v4` with an empty `${{ secrets.CODECOV_TOKEN }}` triggers
`startup_failure`. Fix:

```yaml
tests:
  runs-on: ubuntu-latest
  env:
    CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
  steps:
    # ... test steps ...
    - if: env.CODECOV_TOKEN != ''
      uses: codecov/codecov-action@v4
      with:
        token: ${{ env.CODECOV_TOKEN }}
        fail_ci_if_error: false
```

`secrets` context isn't available in step `if:` per GitHub docs — must
promote to job-level `env`.

---

## Tag discipline during validation

**Do not force-push `v1` during bisection.** Lessons from this session:

1. Test callers pin to commit SHA or explicit prerelease tag like
   `v1.3.0-rc.1`. Example:

   ```yaml
   uses: Artic0din/dev-templates/.github/workflows/ci-core-typescript.yml@v1.3.0-rc.1
   ```

2. Iterate: push commits to `main` + retag `v1.3.0-rc.N` per iteration.
   Each RC is immutable.

3. Move floating `v1` ONLY after all three language workflows validate
   against real callers (PowerBot, PriceHawk, PowerSync, PLNR re-do).

---

## Validation plan

Per-language smoke tests before moving `v1`:

| Language | Test repo | Caller fork-only? |
|---|---|---|
| python | PowerSync fork | yes (caller pins to `@v1.3.0-rc.N`) |
| typescript+pnpm | PowerBot | no |
| typescript+npm | New PLNR scaffold PR | no |
| swift | Amprage or test repo | no |

Each must reach a real `failure` or `success` — not `startup_failure`.

---

## Caller migration

After v1.3.0 ships, existing callers (PowerBot ci.yml, PriceHawk ci.yml,
PowerSync ci.yml, etc.) update their `uses:` line:

```yaml
# Before (v1.2.x):
uses: Artic0din/dev-templates/.github/workflows/ci-core.yml@v1
with:
  language: typescript
  package_manager: pnpm

# After (v1.3.0):
uses: Artic0din/dev-templates/.github/workflows/ci-core-typescript.yml@v1
with:
  package_manager: pnpm
```

Update scaffold-discipline to generate the right `uses:` line based on the
language argument. Add a v1.2 → v1.3 migration helper.

---

## What NOT to do in v1.3.0

- **Don't continue Phase 4 rollout in this session.** PLNR/Arsenal/KiloWasps
  wait for v1.3.0 to land.
- **Don't restore `ci-core.yml` (monolithic).** It stays deleted/renamed.
- **Don't add new features (claude-review gate, version-drift inputs, etc.).**
  v1.3.0 is a structural refactor only.
- **Don't force-push `v1` during debug.** Use RC tags or SHAs.

---

## Acceptance criteria

v1.3.0 ships when:

1. `ci-core-python.yml`, `ci-core-typescript.yml`, `ci-core-swift.yml` exist
   in `.github/workflows/`.
2. Each validates against actionlint.
3. Each calls successfully (reaches real success or failure, not
   startup_failure) from at least one real caller pinned to `@v1.3.0-rc.N`.
4. codecov env-gate carried forward in all three.
5. `scaffold-discipline` updated to emit the right `uses:` per language.
6. `workflow-templates/ci.yml` shows three commented examples (Python, TS,
   Swift).
7. README updated. ROLLOUT-LEDGER updated with v1.3.0 entry.
8. Floating `v1` moved to v1.3.0 final commit.
9. PLNR scaffold PR recreated using v1.3.0 caller — passes the workflow
   validation (reaches real CI results).

---

## Out of scope (defer to v1.4 or later)

- Claude review gate restoration (still has the original startup_failure
  from v1.0.5; needs separate investigation).
- Per-job permissions audit.
- Cost estimation for codecov / Anthropic API.
- Caller migration helper script.

---

End of brief.
