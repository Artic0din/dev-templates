# dev-templates

Discipline stack templates for Ryan's GitHub repos. Three-agent workflow:
Claude Code builds → Codex reviews locally → GitHub Copilot reviews inline on
the PR → CI verifies → Ryan merges.

## Layout

```
dev-templates/
├── .github/
│   ├── workflows/
│   │   ├── ci-core-python.yml       # reusable: Python (uv + ruff + pyright + pytest)
│   │   ├── ci-core-typescript.yml   # reusable: TS (pnpm or npm)
│   │   ├── ci-core-swift.yml        # reusable: Swift (macos runner)
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
| CI | `ci.yml` caller → `ci-core-{language}.yml` | Lint, typecheck, gitleaks, tests, codecov, rollup |
| Ryan | — | Merge manually |

Claude Code Action (gate) is **optional** and currently out of scope (carries
forward a `startup_failure` bug from v1.0.5; v1.4 territory).

## How it's used

Each target repo's `.github/workflows/ci.yml` picks the right per-language
reusable workflow:

```yaml
# Python
jobs:
  core:
    uses: Artic0din/dev-templates/.github/workflows/ci-core-python.yml@v1
    secrets: inherit

# TypeScript (pnpm)
jobs:
  core:
    uses: Artic0din/dev-templates/.github/workflows/ci-core-typescript.yml@v1
    with:
      package_manager: pnpm
    secrets: inherit

# TypeScript (npm + monorepo subdirectory)
jobs:
  core:
    uses: Artic0din/dev-templates/.github/workflows/ci-core-typescript.yml@v1
    with:
      working_directory: nesc-scheduler
      package_manager: npm
    secrets: inherit

# Swift
jobs:
  core:
    uses: Artic0din/dev-templates/.github/workflows/ci-core-swift.yml@v1
    secrets: inherit
```

Each caller adds a `ci-passed` rollup job that branch protection requires.

## Versioning

- `v1.x.y` — specific releases (semver). Immutable tags.
- `v1.x.y-rc.N` — release candidates during validation. Immutable.
- `v1` — floating major tag. Moves only after RC validates against real
  callers. Callers in production repos pin to `@v1`.
- Breaking changes bump to `v2` and require explicit caller migration.

## Scaffold a repo

Three tools, layered:

```bash
discipline-doctor /path/to/repo            # inspect + report (read-only)
discipline-bootstrap /path/to/repo --apply  # doctor + apply scaffold
discipline-bootstrap /path/to/repo --apply --open-pr  # also commit + push + PR
```

Or the low-level copier directly:

```bash
~/bin/scaffold-discipline {python|typescript|swift|node}
```

### `discipline-doctor`

Read-only. Inspects a folder and reports:

- detected language stack (python, typescript, typescript-nextjs, node, swift, …)
- package manager (pnpm / npm / yarn-unsupported / none)
- working_directory hint for monorepos
- git remote + default + current branch
- existing scaffold files (AGENTS.md, CLAUDE.md, workflows)
- CodeRabbit / Sourcery debris
- workflows that may overlap with ci-core
- blockers preventing safe scaffold
- suggested `scaffold-discipline` command

Flags: `--json` (machine-readable for orchestration), `--help`.

Refuses to inspect non-directories. Exits non-zero on blockers.

### `discipline-bootstrap`

Orchestrator. Runs doctor, then optionally applies the scaffold and opens
a draft PR. Refuses to apply if doctor reports any blockers.

### `scaffold-discipline`

Low-level copier. Idempotent — safe to re-run. Migrates v1.0.x repos by
renaming `.github/instructions/*.md` to `*.instructions.md` and stripping
Cursor-only frontmatter fields. Generates a caller `ci.yml` that invokes
the right per-language reusable workflow. Won't create or modify toolchain
config files (`pyproject.toml`, `package.json`, `Package.swift`).

## Pinning policy

- **Allow-list (floating tag OK):** `anthropics/*`, `github/*`, `actions/*`,
  `codecov/*`, `astral-sh/*`, `pnpm/*`. Re-audit quarterly.
- **Everything else: SHA-pin** with version comment.

Reference: tj-actions/changed-files compromise (March 2025) — maintainer
trust is a continuous assessment, not a binary attribute.

## Language toolchain prerequisites

Scaffold does NOT create toolchain config files. Initialise the language
toolchain before running `scaffold-discipline`:

| Language | Prerequisite | Why |
|---|---|---|
| **Python** | `pyproject.toml` + uv | `ci-core-python.yml` runs `uv sync --locked`. Repos with only `requirements.txt` fail CI at the setup step. Migrate via `uv init` + `uv add`. |
| **TypeScript / Node** | `package.json` + `pnpm-lock.yaml` **or** `package-lock.json` | `ci-core-typescript.yml` supports both. Default is pnpm; pass `package_manager: npm` in the caller for npm repos. Yarn not supported — migrate first. |
| **Swift** | `Package.swift` or Xcode project | `ci-core-swift.yml` runs `swiftformat`, `swiftlint`, `swift test`. |

Scaffold warns when these prerequisites are missing.

## Monorepo support

For repos where the project lives in a subdirectory (e.g. PLNR's
`nesc-scheduler/`), set `working_directory` in the caller's `with:` block:

```yaml
core:
  uses: Artic0din/dev-templates/.github/workflows/ci-core-typescript.yml@v1
  with:
    working_directory: nesc-scheduler
    package_manager: npm
```

Default `working_directory: '.'` covers the common case.

## AGENTS.md convention

Two valid shapes:

1. **New repo** — scaffold copies the full template (Codex role + nothing else).
2. **Existing AGENTS.md** (project overview, etc.) — scaffold appends a
   `## Codex Review Role` section if not already present. Existing content
   is preserved. The Codex section can be hand-edited; scaffold only adds
   it if missing on re-run.

This keeps project-description AGENTS.md files (e.g. PriceHawk) intact
while ensuring every repo carries the Codex role brief.

## Codecov env-gate

`codecov-action@v4` with an empty `${{ secrets.CODECOV_TOKEN }}` triggers
GitHub Actions `startup_failure` before any job runs. All ci-core variants
promote the secret to job-level `env` and gate the upload step on
`env.CODECOV_TOKEN != ''`. Repos without the secret silently skip the
upload; repos with it continue uploading.

## Phase 4 rollout order (planned)

1. **PowerSync** (Python, fork) — Python-path validation. Caution re:
   upstream constraints.
2. **PLNR** (TypeScript) — Next.js / Prisma. Will recreate on v1.3.0.
3. **Arsenal** (TypeScript) — React/Vite + Express.
4. **KiloWasps** (TypeScript) — Next.js + SQLite + Recharts. Needs remote first.
5. **Amprage**, **SpoolWise** (Swift) — after TypeScript path settles.

**Excluded** from the standard scaffold:
- **GridWise** — no longer actively developed (clean-slate stub since 2026-05-04).
- **Oversight** (Power BI) — no CI pattern fits.
- **Watchbill** (HTML + Excel/VBA) — no CI pattern fits.
