# apcore-refinery inspect -- Integration Health Report

**Project:** flask-apcore demo (`flask-apcore` v0.3.0)
**Framework:** Flask
**Language:** Python
**Date:** 2026-03-17

| Dimension                | Critical | Warning | Info |
|--------------------------|----------|---------|------|
| D1 Dependency Health     |    0     |    0    |   0  |
| D2 Scanner Config        |    0     |    1    |   1  |
| D3 Output Config         |    0     |    0    |   1  |
| D4 Environment           |    0     |    0    |   0  |
| D5 Annotation Red Flags  |    0     |    7    |   3  |
| D6 Ecosystem Issues      |    0     |    0    |   1  |
|--------------------------|----------|---------|------|
| **TOTAL**                |  **0**   |  **8**  | **5**|

---

## CRITICAL

_(none)_

---

## WARNING

### D2 — Scanner Configuration

**[D2-1]** Found 6 endpoints/modules in code (`list_tasks`, `create_task`, `get_task`, `update_task`, `delete_task`, `task_stats`) but only 5 `.binding.yaml` files. The `@module`-decorated `task_stats.v1` function has no binding YAML.
- Fix: Run the scanner again, or create `apcore_modules/task_stats.v1.binding.yaml` manually.

### D5 — Annotation Red Flags

**[D5-1]** `list_tasks.get`: GET endpoint has `idempotent: false` -- GET requests are inherently idempotent (same request = same response, no side effects).
- Fix: Set `idempotent: true`.

**[D5-2]** `get_task.get`: GET endpoint has `idempotent: false` -- GET requests are inherently idempotent.
- Fix: Set `idempotent: true`.

**[D5-3]** `delete_task.delete`: `destructive: true` but `requires_approval: false` -- permanent data loss without human confirmation.
- Fix: Set `requires_approval: true`.

**[D5-4]** `list_tasks.get`: `open_world: true` but function body contains no external calls (no HTTP, DB, file I/O, or subprocess). The `list_tasks()` function only iterates over the in-memory `_tasks` dict and returns Pydantic model instances.
- Fix: Set `open_world: false`.

**[D5-5]** `create_task.post`: `open_world: true` but function body contains no external calls. The `create_task()` function only manipulates the in-memory `_tasks` dict and `_next_id` counter.
- Fix: Set `open_world: false`.

**[D5-6]** `get_task.get`: `open_world: true` but function body contains no external calls. The `get_task()` function only reads from the in-memory `_tasks` dict.
- Fix: Set `open_world: false`.

**[D5-7]** `update_task.put`: `open_world: true` but function body contains no external calls. The `update_task()` function only mutates entries in the in-memory `_tasks` dict.
- Fix: Set `open_world: false`.

**[D5-8]** `delete_task.delete`: `open_world: true` but function body contains no external calls. The `delete_task()` function only deletes from the in-memory `_tasks` dict.
- Fix: Set `open_world: false`.

---

## INFO

### D2 — Scanner Configuration

**[D2-2]** No `APCORE_INCLUDE_PATHS` / `APCORE_EXCLUDE_PATHS` patterns configured; all endpoints will be scanned.

### D3 — Output Configuration

**[D3-1]** No explicit `--output yaml` or `APCORE_OUTPUT_FORMAT` setting found. The project uses `APCORE_MODULE_DIR="apcore_modules/"` which implies YAML binding mode is active (binding files are present). No action needed, but consider setting `APCORE_OUTPUT_FORMAT=yaml` explicitly for clarity.

### D5 — Annotation Red Flags

**[D5-9]** `list_tasks.get`: `cacheable: true` but `cache_ttl: 0`. Either set a meaningful TTL or set `cacheable: false`.

**[D5-10]** `get_task.get`: `cacheable: true` but `cache_ttl: 0`. Either set a meaningful TTL or set `cacheable: false`.

**[D5-11]** `list_tasks.get`: `paginated: false` but `pagination_style: cursor` is set. This is contradictory -- remove `pagination_style` or set `paginated: true`.

---

## Detailed Analysis

### Step 0: Project Detection

- Build config: Parent `pyproject.toml` at `/Users/tercelyi/Workspace/aipartnerup/flask-apcore/pyproject.toml`
- Integration package: `flask-apcore` v0.3.0
- Framework: Flask
- Language: Python (requires-python >= 3.11)
- The demo directory itself has no `pyproject.toml` or `requirements.txt`; it relies on the parent project.

### Step 1: D1 -- Dependency Health

- **apcore SDK**: Required via `apcore>=0.13.0` in `pyproject.toml` -- present.
- **apcore-toolkit**: Required via `apcore-toolkit>=0.2.0` -- present and not outdated (>= 0.2.0).
- **Pydantic**: Required via `pydantic>=2.0` -- present.
- **Version compatibility**: `flask-apcore` declares `apcore>=0.13.0`, no conflicts detected in declared dependencies.
- No issues found.

### Step 2: D2 -- Scanner Configuration

- **Scanner entry point**: `Apcore(app)` found at line 159 of `app.py` -- scanner is configured.
- **Schema backend**: Pydantic is a declared dependency (`pydantic>=2.0`) -- present.
- **Include/exclude patterns**: Not configured (info-level observation).
- **Endpoint coverage**: 6 endpoints/modules in code, 5 binding YAML files. The `task_stats.v1` standalone `@module` function is missing a binding YAML.

### Step 3: D3 -- Output Configuration

- `APCORE_MODULE_DIR="apcore_modules/"` is configured in `app.py`.
- The `apcore_modules/` directory exists and contains 5 `.binding.yaml` files.
- No explicit output format setting, but YAML bindings are present and functional.

### Step 4: D4 -- Environment & Settings

- `.env.example` reviewed: no deprecated `APCORE_AI_ENABLED=true` setting.
- `APCORE_SCAN_AI_ENHANCE=false` is commented out in `.env.example` -- no issue.
- `APCORE_MODULE_DIR` in app config points to `apcore_modules/` which exists.
- No `APCORE_DEBUG=true` in production config (the file is an example).
- No issues found.

### Step 5: D5 -- Annotation Red Flags

Detailed function body analysis for `open_world` checks:

| Module | Target | Function Body | External Calls? | open_world | Verdict |
|--------|--------|---------------|-----------------|------------|---------|
| `list_tasks.get` | `app:list_tasks` (line 106) | Iterates `_tasks.values()`, returns list of `Task` models | None | `true` | WRONG |
| `create_task.post` | `app:create_task` (line 112) | Reads `body`, writes to `_tasks` dict, increments `_next_id` | None | `true` | WRONG |
| `get_task.get` | `app:get_task` (line 122) | Reads `_tasks.get(task_id)` | None | `true` | WRONG |
| `update_task.put` | `app:update_task` (line 131) | Reads `_tasks.get(task_id)`, mutates dict fields | None | `true` | WRONG |
| `delete_task.delete` | `app:delete_task` (line 146) | Checks `task_id in _tasks`, does `del _tasks[task_id]` | None | `true` | WRONG |

All five scanned modules have `open_world: true` but none of their function bodies contain any external call patterns (`requests.`, `httpx.`, `urllib`, `aiohttp`, `session.`, `.query(`, `.execute(`, `open(`, `Path(`, `subprocess.`, `os.system`). Every function only manipulates the in-memory `_tasks` dictionary. All should be `open_world: false`.

Additional annotation checks:
- **GET + idempotent:false**: `list_tasks.get` and `get_task.get` both have `idempotent: false` despite being GET endpoints.
- **destructive + requires_approval**: `delete_task.delete` has `destructive: true` but `requires_approval: false`.
- **cacheable + cache_ttl:0**: `list_tasks.get` and `get_task.get` are `cacheable: true` with `cache_ttl: 0`.
- **paginated:false + pagination_style set**: `list_tasks.get` has `paginated: false` but `pagination_style: cursor`. (Note: other modules also have `pagination_style: cursor` with `paginated: false`, but `list_tasks` is the only one where pagination is semantically relevant, so only flagging that one.)

### Step 6: D6 -- Known Ecosystem Issues

- **apcore-a2a**: Not in declared dependencies -- A2A annotation loss check not applicable.
- **RegistryWriter schema passthrough**: The project uses YAML binding mode (`APCORE_MODULE_DIR` set, binding files present), so the RegistryWriter passthrough bug does not apply.
- **apcore-mcp**: Listed as an optional dependency (`apcore-mcp>=0.10.0` under `[mcp]` extra). If installed: `info` -- apcore-mcp does not include module examples in MCP Tool definitions.

---

## Recommendation

Fix the 8 warnings before running `/apcore-refinery:score`. The most impactful fixes are:
1. Set `open_world: false` on all 5 modules (they are purely in-memory operations).
2. Set `idempotent: true` on the 2 GET endpoints.
3. Set `requires_approval: true` on `delete_task.delete`.

These annotation errors would mislead LLM agents about the tool's behavior -- `open_world: true` maps to MCP's `openWorldHint` and incorrectly signals that these tools interact with external systems.
