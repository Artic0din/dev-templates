---
applyTo: "tests/**"
---

# Test conventions (pytest + pytest-homeassistant-custom-component)

- `pytest-asyncio` is auto mode — async test functions just work, no decorator needed.
- Use the `hass` fixture from `pytest_homeassistant_custom_component` to spin up a Home Assistant instance.
- Use `MockConfigEntry` from `pytest_homeassistant_custom_component.common` to seed config entries.
- Mock external HTTP via `aioresponses` or `aiohttp_client_session_mocker`. Never hit real APIs in tests.
- Each integration entry-point gets a smoke test: `async_setup_entry → True` + `async_unload_entry → True`.
- Coordinator tests: assert `_async_update_data` populates the expected shape; mock the upstream client.
- Use `caplog` to assert log messages, not stdout.
- `S101` (assert) is allowed in tests — see `pyproject.toml` per-file-ignores.
