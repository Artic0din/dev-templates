# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Historical detail for v1.0.0 – v1.3.0 lives in [ROLLOUT-LEDGER.md](ROLLOUT-LEDGER.md).

## [Unreleased]

### Added

- **`template-ha-component` expansion** (PR #8). Eight new managed files:
  - `.github/instructions/python.instructions.md` (path-scoped `**/*.py`)
  - `.github/instructions/tests.instructions.md` (path-scoped `tests/**`)
  - `.github/instructions/home-assistant.instructions.md` (path-scoped `custom_components/**`)
  - `.github/copilot-instructions.md` (thin forwarder to AGENTS.md)
  - `.github/dependabot.yml` (pip + github-actions w/ grouping per v3 §14)
  - `codecov.yml` (auto target + 80% patch + `require_changes: true`)
  - `tests/conftest.py.jinja` (auto-enable custom integrations fixture)
  - `tests/test_init.py.jinja` (smoke test: `async_setup_entry` + `async_unload_entry`)
- **`template-python` skeleton** (PR #9). Generic Python project template:
  `AGENTS.md.jinja`, `README.md.jinja`, `pyproject.toml.jinja` (hatchling +
  uv + ruff + pyright + pytest, strict pyright, target-version derived from
  `python_version` answer), `.pre-commit-config.yaml`, `.gitignore`,
  `src/{{ python_module }}/__init__.py.jinja`, `tests/__init__.py`,
  `tests/test_smoke.py.jinja`, `.github/workflows/ci.yml.jinja` calling
  `ci-core-python.yml@v2`, `.github/instructions/{python,tests}.instructions.md`,
  `.github/copilot-instructions.md` (thin forwarder), `.github/dependabot.yml`
  (pip + actions grouping), `codecov.yml`.
- New `python_module` question in root `copier.yml`, defaulted to
  `{{ project_slug | replace('-', '_') }}`, gated to `template-python`.

### Changed

- **`scripts/scaffold-discipline` rewritten as a thin Copier wrapper** (~63 lines vs ~257
  in v1.3.0). Picks one of the four `template-*` subdirectories and calls
  `uvx --from "copier>=9.5" copier copy --trust --vcs-ref v2 ...` against
  `https://github.com/Artic0din/dev-templates.git`. Refuses non-empty destinations
  (use `copier update` for existing repos). `swift-ios` template intentionally blocked
  pending a 2nd Swift project. Overridable via `SCAFFOLD_TEMPLATES_REF` /
  `SCAFFOLD_TEMPLATES_REPO` env vars for testing.

### Removed

- v1.0.x → v1.1.0 file-rename migration logic (v1.3.0 is now the floor).
- Per-language toolchain prerequisite warnings (moved to Copier `_message_after_copy`
  and per-template `pyproject.toml.jinja` defaults).
- File-by-file copy / overwrite logic (Copier owns it now).

### Docs

- README "Scaffold a repo" section rewritten for the v2 Copier flow, including
  the `copier update` retrofit path and a fallback pointer to the `v1.3.0` tag
  for unmigrated repos.

## [v2.0.0] — 2026-05-25

### Added

- **v2 Copier scaffolding** (PR #6). Root `copier.yml` with subdirectory dispatch,
  4-stack choice (`template-ha-component`, `template-python`, `template-ts-node`,
  `template-swift-ios`), and Option A `_message_after_copy` checklist for post-scaffold
  `gh repo create` + `gh label create ai-review-override`.
- **`template-ha-component/` skeleton** (PR #6): `AGENTS.md.jinja` with Codex P0/P1
  review guidelines, `README.md.jinja`, `manifest.json.jinja`, `__init__.py.jinja`,
  `const.py.jinja`, `pyproject.toml.jinja` (uv + ruff + pyright + pytest),
  `.pre-commit-config.yaml`, `ci.yml.jinja` calling `ci-core-python.yml@v2`,
  `hassfest.yml`, `hacs.yml`, `hacs.json.jinja`, `.gitignore`.
- `_tasks` for `git init -b main` + `CLAUDE.md → AGENTS.md` symlink + `uv sync` + `pre-commit install`.
- `_skip_if_exists` protects user-edited `AGENTS.md`, `README.md`, `pyproject.toml`,
  `package.json`, `Package.swift`, `.env*` from being clobbered on `copier update`.

### Targeted migration

- **ha-pricehawk** (HIGH active) → pilot via `copier copy` after merge + `v2.0.0` tag.
- **PowerSync** (fork) → AGENTS.md-only treatment, no scaffold injection (upstream rejects fork-only CI).

### Out of scope for this PR (follow-ups)

- `scripts/scaffold-discipline` rewrite as Copier wrapper (Patch 1 — requires `v2.0.0` tag).
- `template-python`, `template-ts-node` (Phase 6 of v3 playbook).
- `template-swift-ios` (deferred — only one Swift repo).
- `deploy-live.yml.jinja` with fork guard (Patch 4 / Phase 8).
- `tests/`, `.github/instructions/*.instructions.md`, `dependabot.yml`, `codecov.yml` for the HA template.

## Released

See [ROLLOUT-LEDGER.md](ROLLOUT-LEDGER.md) for v1.0.0 through v1.3.0 history.
