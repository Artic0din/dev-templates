# dev-templates

Discipline stack templates for Ryan's GitHub repos. v1.1.0 adopts the
**three-agent workflow**: Claude Code builds → Codex reviews locally →
GitHub Copilot reviews inline on the PR → CI verifies → Ryan merges.

## Layout

```
dev-templates/
├── .github/
│   ├── workflows/
│   │   ├── ci-core.yml              # reusable workflow (callers invoke)
│   │   ├── security-scan.yml        # gitleaks + dependency-review
│   │   ├── docs-check.yml           # CHANGELOG modified gate
│   │   ├── pr-title-check.yml       # conventional commit types
│   │   └── version-drift-guard.yml  # multi-file version consistency
│   ├── instructions/                # *.instructions.md per language + topic
│   │   ├── meta.instructions.md
│   │   ├── code-review.instructions.md
│   │   ├── documentation.instructions.md
│   │   ├── tests.instructions.md
│   │   ├── python.instructions.md
│   │   ├── typescript.instructions.md
│   │   ├── swift.instructions.md
│   │   └── node.instructions.md
│   └── prompts/
│       └── pr-review.prompt.md      # Claude review prompt (optional)
├── workflow-templates/
│   └── ci.yml                       # caller template (editable in repos)
├── templates/
│   ├── AGENTS.md                    # Codex local review role
│   ├── CLAUDE.md                    # Claude Code builder discipline
│   ├── CONTRIBUTING.md
│   └── copilot-instructions.md      # Copilot inline review steering
├── configs/
│   ├── codecov.yml
│   ├── .gitleaks.toml
│   └── zensical.toml
└── scripts/
    ├── scaffold-discipline          # scaffold + migration script
    └── carl-test                    # CARL JSON domain trigger debug
```

## Agent roles

| Agent | File | Role |
|---|---|---|
| Claude Code | `CLAUDE.md` | Build the change, local checks, draft PR |
| Codex | `AGENTS.md` | Local review pre-PR: bugs, security, missing tests, scope creep |
| GitHub Copilot | `.github/copilot-instructions.md` | Inline review on the PR (post-push) |
| CI | `ci.yml` caller → `ci-core.yml` | Lint, typecheck, gitleaks, tests, codecov, rollup |
| Ryan | — | Merge manually |

Claude Code Action (gate) is **optional** in v1.1.0 and currently disabled
pending a startup_failure bug in GitHub Actions when the gate uses
`if: always()` / `if: !cancelled()` together with `needs:`.

## How it's used

Each target repo's `.github/workflows/ci.yml` calls the reusable workflow:

```yaml
jobs:
  core:
    uses: Artic0din/dev-templates/.github/workflows/ci-core.yml@v1
    with:
      language: typescript
    secrets: inherit

  ci-passed:
    name: CI passed
    if: always()
    needs: [core]
    runs-on: ubuntu-latest
    steps:
      - run: '[[ "${{ needs.core.result }}" == "success" ]] || exit 1'
```

Branch protection on `main` requires the literal `CI passed` check.

## Versioning

- `v1.x.y` — specific releases (semver).
- `v1` — floating major tag. Callers pin to this.
- Breaking changes bump to `v2` and require explicit caller update.

## Scaffold a repo

```bash
~/bin/scaffold-discipline {python|typescript|swift|node}
```

From inside the target repo. Idempotent — safe to re-run. Migrates v1.0.x
repos by renaming `.github/instructions/*.md` to `*.instructions.md` and
stripping Cursor-only frontmatter fields.

## Pinning policy

- **Allow-list (floating tag OK):** `anthropics/*`, `github/*`, `actions/*`, `codecov/*`, `astral-sh/*`, `pnpm/*`. Re-audit quarterly.
- **Everything else: SHA-pin** with version comment.

Reference: tj-actions/changed-files compromise (March 2025) — maintainer
trust is a continuous assessment, not a binary attribute.

## Language toolchain prerequisites

Scaffold does NOT create toolchain config files (per Clarification G).
Initialise the language toolchain before running `scaffold-discipline`:

| Language | Prerequisite | Why |
|---|---|---|
| **Python** | `pyproject.toml` + uv | ci-core Python jobs use `uv sync --locked`. Repos with only `requirements.txt` will fail CI at the setup step. Migrate via `uv init` + `uv add`. |
| **TypeScript / Node** | `package.json` + `pnpm-lock.yaml` **or** `package-lock.json` | ci-core supports both. Default is pnpm; set `package_manager: npm` in caller for npm repos. Yarn not supported — migrate to pnpm or npm. |
| **Swift** | `Package.swift` or Xcode project | ci-core runs `swiftformat`, `swiftlint`, `swift build`, `swift test`. |

Scaffold warns when these prerequisites are missing.

### Monorepo support (v1.2.0+)

For repos where the project lives in a subdirectory (e.g. PLNR's `nesc-scheduler/`),
set `working_directory` in the caller's `with:` block:

```yaml
core:
  uses: Artic0din/dev-templates/.github/workflows/ci-core.yml@v1
  with:
    language: typescript
    working_directory: nesc-scheduler
    package_manager: npm
```

Defaults: `working_directory: '.'`, `package_manager: 'pnpm'`. Both inputs are
backward-compatible — existing callers (PowerBot, PriceHawk, PowerSync) keep
working without changes.

## AGENTS.md convention

Two valid shapes:

1. **New repo** — scaffold copies the full template (Codex role + nothing else).
2. **Existing AGENTS.md** (project overview, etc.) — scaffold appends a
   `## Codex Review Role` section if not already present. Existing content
   is preserved. The Codex section can be hand-edited; scaffold only adds
   it if missing on re-run.

This keeps project-description AGENTS.md files (e.g. PriceHawk) intact
while ensuring every repo carries the Codex role brief.

## Phase 4 rollout order (planned)

1. **PowerSync** (Python, fork) — Python-path validation. Caution re: upstream constraints.
2. **PLNR** (TypeScript) — Next.js / Prisma.
3. **Arsenal** (TypeScript) — React/Vite + Express.
4. **KiloWasps** (TypeScript) — Next.js + SQLite + Recharts. Needs remote first.
5. **Amprage**, **SpoolWise** (Swift) — after TypeScript path settles.

**Excluded** from the standard scaffold:
- **GridWise** — no longer actively developed (clean-slate stub since 2026-05-04).
- **Oversight** (Power BI) — no CI pattern fits.
- **Watchbill** (HTML + Excel/VBA) — no CI pattern fits.
