# dev-templates

Discipline stack templates for Ryan's GitHub repos. Source of truth for:

- **`.github/workflows/ci-core.yml`** — reusable CI workflow callers invoke.
- **`workflow-templates/ci.yml`** — caller template (editable per repo).
- **`.github/instructions/`** — repo-rule templates for AI reviewers.
- **`.github/prompts/pr-review.prompt.md`** — Claude review prompt.
- **`configs/`** — codecov.yml, .gitleaks.toml, zensical.toml.
- **`templates/CONTRIBUTING.md`** — contributor guide template.
- **`scripts/scaffold-discipline`** — scaffold script for new/existing repos.

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
      - run: |
          [[ "${{ needs.core.result }}" == "success" ]] || exit 1
```

Branch protection on `main` requires the literal `CI passed` check.

## Versioning

- `v1.0.0`, `v1.0.1`, ... — specific releases.
- `v1` — floating major tag. Callers pin to this.
- Breaking changes bump to `v2` and require explicit caller update.

## Scaffold a new repo

```bash
~/bin/scaffold-discipline {python|typescript|swift|node}
```

Run from inside the target repo. Idempotent — safe to re-run after
template updates.

## Pinning policy

- **Allow-list (floating tag OK):** `anthropics/*`, `github/*`,
  `actions/*`, `codecov/*`, `astral-sh/*`, `pnpm/*`. Re-audit quarterly.
- **Everything else: SHA-pin** with version comment.

Reference: tj-actions/changed-files compromise (March 2025) — maintainer
trust is a continuous assessment.
