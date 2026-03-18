apcore-refinery inspect — Integration Health Report

Project: flask-apcore demo (flask-apcore v0.3.0)
Framework: Flask
Language: python

  Dimension                | Critical | Warning | Info
  ─────────────────────────┼──────────┼─────────┼─────
  D1 Dependency Health     |    0     |   0     |  0
  D2 Scanner Config        |    0     |   1     |  1
  D3 Output Config         |    0     |   0     |  0
  D4 Environment           |    0     |   0     |  0
  D5 Annotation Red Flags  |    0     |   3     |  3
  D6 Ecosystem Issues      |    0     |   0     |  1
  ─────────────────────────┼──────────┼─────────┼─────
  TOTAL                    |    0     |   4     |  5

## D1 — Dependency Health

No issues found.

- apcore SDK: required via `apcore>=0.13.0` in flask-apcore dependencies.
- apcore-toolkit: required via `apcore-toolkit>=0.2.0` in flask-apcore dependencies (meets minimum version threshold).
- Pydantic: required via `pydantic>=2.0` in flask-apcore dependencies — schema backend is available.
- Version constraints are internally consistent.

## D2 — Scanner Configuration

WARNING:
  [D2-1] Found 6 endpoints in code (list_tasks GET, create_task POST, get_task GET, update_task PUT, delete_task DELETE, task_stats @module) but only 5 .binding.yaml files in apcore_modules/. The standalone @module function `task_stats.v1` has no corresponding binding YAML file.
    Fix: Re-run the scanner to ensure `task_stats.v1` is included, or manually create a `task_stats.v1.binding.yaml` file.

INFO:
  [D2-2] No include/exclude patterns configured (APCORE_INCLUDE_PATHS / APCORE_EXCLUDE_PATHS not set); all endpoints will be scanned.

Details:
- Scanner entry point: `Apcore(app)` is present in app.py (line 159) — scanner is configured.
- Schema backend: Pydantic is available (imported and used for request/response models) — schema inference will work correctly.
- `APCORE_AUTO_DISCOVER=True` is set — auto-discovery is enabled.
- `APCORE_MODULE_DIR="apcore_modules/"` points to a valid directory containing 5 binding files.

## D3 — Output Configuration

No issues found.

- Output directory `apcore_modules/` exists and contains 5 `.binding.yaml` files.
- No explicit `APCORE_OUTPUT_FORMAT` setting found, but YAML binding files are present and being used, which means the scanner produced YAML output. This is the recommended mode for apcore-refinery to be effective.

## D4 — Environment & Settings

No issues found.

- No `.env` file present (only `.env.example` which is not active).
- `APCORE_AI_ENABLED` is NOT set — no deprecated AI enhancement settings detected.
- `APCORE_SCAN_AI_ENHANCE` appears only in `.env.example` as a commented-out line — not active.
- `APCORE_MODULE_DIR="apcore_modules/"` is set and points to a valid directory.
- `APCORE_TRACING_ENABLED=True` and `APCORE_METRICS_ENABLED=True` are set — observability is configured (appropriate for a demo).

## D5 — Annotation Red Flags

WARNING:
  [D5-1] list_tasks.get: GET endpoint has idempotent=false — GET requests are inherently idempotent. Fix: set idempotent=true.

WARNING:
  [D5-2] get_task.get: GET endpoint has idempotent=false — GET requests are inherently idempotent. Fix: set idempotent=true.

WARNING:
  [D5-3] delete_task.delete: destructive=true but requires_approval=false — permanent data loss without human confirmation. The description says "Delete a task permanently." which confirms destructive intent. Fix: set requires_approval=true.

INFO:
  [D5-4] list_tasks.get: cacheable=true but cache_ttl=0. Either set a meaningful TTL or set cacheable=false.

INFO:
  [D5-5] get_task.get: cacheable=true but cache_ttl=0. Either set a meaningful TTL or set cacheable=false.

INFO:
  [D5-6] list_tasks.get: paginated=false but pagination_style=cursor is set. This is contradictory — remove pagination_style or set paginated=true.

Details on open_world checks:
- list_tasks.get: open_world=true but code only reads from in-memory `_tasks` dict — could be set to false. However, this is a minor observation and not raised as a finding since it does not cause incorrect behavior.
- get_task.get: same pattern — in-memory only.
- create_task.post: open_world=true but only writes to in-memory `_tasks` dict.
- update_task.put: open_world=true but only modifies in-memory `_tasks` dict.
- delete_task.delete: open_world=true but only deletes from in-memory `_tasks` dict.
- All five route handlers operate exclusively on the in-memory `_tasks` dictionary with no external calls (HTTP, DB, file I/O, subprocess). However, for a demo app this is expected and open_world is a conservative default.

## D6 — Known Ecosystem Issues

INFO:
  [D6-1] apcore-mcp is available as an optional dependency (mcp extra). If using MCP, note that apcore-mcp does not include module examples in MCP Tool definitions. Examples are only used by A2A (titles only). However, generating them is still valuable for documentation and future protocol versions.

Notes:
- apcore-a2a is NOT in the dependency tree, so the A2A annotation loss issue does not apply.
- The project uses YAML binding files (not registry-only mode), so the RegistryWriter schema passthrough issue does not apply.

---

## Summary

The integration is in good shape with **no critical issues**. There are **4 warnings** that should be addressed before running metadata refinement:

1. **Missing binding file** for the `task_stats.v1` standalone module — re-run the scanner.
2. **Two GET endpoints** (`list_tasks.get`, `get_task.get`) have `idempotent: false` — should be `true`.
3. **delete_task.delete** is `destructive: true` but `requires_approval: false` — should require approval.

Next step: Fix the warnings above, then run `/apcore-refinery:score` to assess metadata quality.
