---
applyTo: "**/*.py"
---

# Python conventions

- Type hints on every public function and class. `from __future__ import annotations` at top of every file.
- `_LOGGER = logging.getLogger(__name__)` per module. Never `print()` in committed code.
- Catch specific exceptions. Never bare `except:` / `except Exception:`.
- Constants module-level. No magic numbers in business logic.
- Prefer `pathlib.Path` over string paths.
- Public API gets a docstring. Internal helpers don't need one if the name is self-describing.
- Async-only when the workload requires it. Don't `async def` for code that doesn't await.
