---
applyTo: "**/*.py"
---

# Python conventions (HA integration)

- Type hints on every public function and class. `from __future__ import annotations` at top of every file.
- `_LOGGER = logging.getLogger(__name__)` per module. Never `print()`.
- Async-first. No blocking I/O in the HA event loop.
  HTTP via `homeassistant.helpers.aiohttp_client.async_get_clientsession(hass)`.
- DataUpdateCoordinator pattern for polling integrations.
- `@callback` decorator on any sync function called from async context (HA dispatcher signals, etc.).
- Catch specific exceptions (`aiohttp.ClientError`, `asyncio.TimeoutError`, vendor classes).
  Never bare `except:` / `except Exception:`. Re-raise `UpdateFailed` from coordinator handlers.
- Constants in `const.py`. No magic numbers in business logic.
- Validate config entry data with `voluptuous` schemas on setup.
- Use `homeassistant.const.UnitOfPower` / `UnitOfEnergy` / etc. for entity units. Don't hardcode `"W"` / `"kWh"` strings.
