---
applyTo: "tests/**"
---

# Test conventions

- `pytest` style, not `unittest.TestCase`.
- Test functions: `def test_<thing>_<scenario>() -> None:`.
- One assertion focus per test. Multiple `assert`s OK if they're about the same invariant.
- Mock external HTTP via `httpx.MockTransport` or `responses`. Never hit real APIs.
- Use `tmp_path` / `tmp_path_factory` for filesystem tests.
- Use `caplog` to assert log output, not stdout capture.
- `S101` (assert) is allowed in tests — see `pyproject.toml` per-file-ignores.
