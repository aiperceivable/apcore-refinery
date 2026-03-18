# apcore-refinery inspect — Integration Health Report

**Project:** flask-apcore demo (`flask-apcore` v0.3.0)
**Framework:** Flask
**Language:** Python
**Path:** `/Users/tercelyi/Workspace/aipartnerup/flask-apcore/examples/demo/`
**Date:** 2026-03-17

---

## Summary

| Dimension                | Critical | Warning | Info |
|--------------------------|----------|---------|------|
| D1 Dependency Health     |    0     |    0    |   0  |
| D2 Scanner Config        |    0     |    1    |   1  |
| D3 Output Config         |    0     |    0    |   0  |
| D4 Environment           |    0     |    0    |   0  |
| D5 Ecosystem Issues      |    0     |    1    |   1  |
|--------------------------|----------|---------|------|
| **TOTAL**                |  **0**   | **2**   | **2**|

---

## D1 — Dependency Health

No issues found.

- `apcore>=0.13.0` is declared as a dependency of `flask-apcore` in `pyproject.toml`.
- `apcore-toolkit>=0.2.0` is declared as a dependency of `flask-apcore` in `pyproject.toml`.
- `pydantic>=2.0` is declared as a dependency (schema backend).
- Version constraints are consistent. `flask-apcore` v0.3.0 requires `apcore>=0.13.0` and `apcore-toolkit>=0.2.0`, which are current.

---

## D2 — Scanner Configuration

### [D2-1] WARNING: Endpoint coverage gap — `task_stats.v1` standalone module has no .binding.yaml

Found 6 callable endpoints in code (5 Flask routes + 1 `@module` decorated function `task_stats`), but only 5 `.binding.yaml` files in `apcore_modules/`:

| Code endpoint     | Binding file                            | Status  |
|-------------------|-----------------------------------------|---------|
| `list_tasks`      | `list_tasks.get.binding.yaml`           | Present |
| `create_task`     | `create_task.post.binding.yaml`         | Present |
| `get_task`        | `get_task.get.binding.yaml`             | Present |
| `update_task`     | `update_task.put.binding.yaml`          | Present |
| `delete_task`     | `delete_task.delete.binding.yaml`       | Present |
| `task_stats`      | —                                       | **Missing** |

**Fix:** Re-run the scanner (`flask apcore scan --output yaml --dir apcore_modules`) to generate the missing binding for `task_stats.v1`. Alternatively, the `@module` decorator may register it directly in-memory, in which case this is expected — but having a YAML binding allows apcore-refinery to score and refine its metadata.

### [D2-2] INFO: No include/exclude patterns configured

Neither `APCORE_INCLUDE_PATHS` nor `APCORE_EXCLUDE_PATHS` is set. All endpoints will be scanned. This is fine for a demo project.

---

## D3 — Output Configuration

No issues found.

- YAML output mode is configured in `entrypoint.sh` via `flask apcore scan --output yaml --dir /app/apcore_modules`.
- `APCORE_MODULE_DIR` is set to `apcore_modules/` in `app.py` (line 62).
- The `apcore_modules/` directory exists and contains 5 `.binding.yaml` files.

---

## D4 — Environment & Settings

No issues found.

- No `.env` file is present (only `.env.example`). All active configuration is in `app.py` via `app.config.update()`.
- `APCORE_SCAN_AI_ENHANCE` appears commented out in `.env.example` — not active.
- `APCORE_AI_ENABLED` is only referenced in `README.md` as documentation — not active in any config.
- `APCORE_MODULE_DIR="apcore_modules/"` points to an existing directory.
- `APCORE_TRACING_ENABLED`, `APCORE_METRICS_ENABLED`, `APCORE_LOGGING_ENABLED` are set in `app.py` — observability is configured.
- No contradictory settings detected.

---

## D5 — Known Ecosystem Issues

### [D5-1] WARNING: RegistryWriter schema passthrough limitation

The demo app uses both YAML output mode (via `entrypoint.sh`) and in-memory auto-discovery (`APCORE_AUTO_DISCOVER=True` in `app.py`). When running locally without the entrypoint script, the app may fall back to in-memory registry mode.

In registry mode, `RegistryWriter` does **not** pass `input_schema`/`output_schema` to `FunctionModule`. `FunctionModule` always re-generates schemas from function signatures, ignoring any YAML-level refinements. If you refine schemas in the `.binding.yaml` files using apcore-refinery, make sure the app loads from YAML (via `BindingLoader` / `APCORE_MODULE_DIR`) rather than relying on in-memory registry alone.

**Fix:** When running locally, ensure the scanner has been run and `APCORE_MODULE_DIR` is set so YAML bindings are loaded. The current `app.py` already sets `APCORE_MODULE_DIR="apcore_modules/"`, so this should work as long as the binding files are present.

### [D5-2] INFO: apcore-mcp does not include module examples in MCP Tool definitions

`apcore-mcp>=0.10.0` is listed as an optional dependency (under `[project.optional-dependencies] mcp`). The demo `app.py` configures MCP server settings (transport, host, port). Note that `apcore-mcp` does not include module `examples` in MCP Tool definitions — examples are only used by A2A (titles only). However, generating examples is still valuable for documentation and future protocol versions.

---

## Verdict

No critical issues found. The integration is in good shape for a demo project. Two warnings and two informational notes were identified.

**Recommended next steps:**

1. **Re-run the scanner** to generate a `.binding.yaml` for the `task_stats.v1` standalone module (or confirm it is intentionally in-memory only).
2. **Proceed to** `/apcore-refinery:score` to assess the metadata quality of the 5 existing binding files.
