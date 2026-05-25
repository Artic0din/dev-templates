---
applyTo: "custom_components/**"
---

# Home Assistant integration conventions

- `manifest.json` keys live in a stable order: `domain, name, version, documentation, issue_tracker, codeowners, config_flow, iot_class, integration_type, requirements, dependencies`.
- `version` follows semver. Bump on every release that ships to HACS.
- `iot_class` reflects polling/push + cloud/local accurately. HACS / hassfest enforce this.
- `config_flow: true` requires a `config_flow.py` exporting `ConfigFlow` subclass.
- Coordinator per device or per account, never per platform.
- Entity classes inherit from `CoordinatorEntity` (DataUpdateCoordinator pattern).
- Unique IDs use the device + entity_type concat: `f"{device_id}_{entity_type}"`. Never use entity_id as unique_id.
- Service registrations: domain-scoped, document arg schema in `services.yaml`.
- Diagnostics: implement `diagnostics.py` with `async_get_config_entry_diagnostics` to redact secrets.
- Repairs: implement `repairs.py` for user-actionable error conditions (expired tokens, missing devices).
- Translation strings live in `strings.json` (English) + `translations/<lang>.json`. Don't hardcode user-facing strings.
- Deploy: SCP `custom_components/<domain>/` to live HA + POST `/api/config/config_entries/entry/{id}/reload` via HA REST API. See `.github/workflows/deploy-live.yml`.
