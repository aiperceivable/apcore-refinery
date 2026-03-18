---
name: inspect
description: >
  Inspect and diagnose the health of a *-apcore project integration. Use this skill
  when the user wants to check if their integration is set up correctly, verify
  dependencies and versions, audit scanner configuration, check annotation correctness
  (idempotent, destructive, open_world, requires_approval), or detect known ecosystem
  issues (A2A annotation loss, RegistryWriter schema passthrough). Covers flask-apcore,
  django-apcore, nestjs-apcore, tiptap-apcore. Reports findings with severity levels
  (critical/warning/info) across 6 dimensions. Supports --fix for safe auto-remediations.
---

# Apcore Refinery — Inspect

Check whether a `*-apcore` integration is set up correctly before worrying about metadata quality.

## Iron Law

**Diagnose before prescribing.** Inspect finds problems in the integration setup itself — dependency mismatches, missing config, uncovered endpoints, and known ecosystem bugs. These structural issues should be fixed before spending time on metadata refinement.

## Why This Skill Exists (Not Just "Check Dependencies")

Beyond basic dependency checks, inspect detects **ecosystem-level issues** that affect metadata quality in ways that are invisible to the user:

- The **A2A bridge has dead code** (`_build_extensions()` defined but never called) — all annotations are lost for A2A consumers. Users need to know this so they embed safety signals in descriptions.
- The **RegistryWriter doesn't pass `input_schema`/`output_schema`** to FunctionModule — any schema refinement via YAML is lost when using Registry mode. Users need to use YAML output mode for refinery to be effective.
- **Framework-specific scanner misconfigurations** (Flask: missing Pydantic backend, NestJS: missing @ApTool decorators) cause silent schema degradation.

## Command Format

```
/apcore-refinery:inspect [path] [--fix]
```

| Flag | Default | Description |
|------|---------|-------------|
| `path` | CWD | Directory of the *-apcore project |
| `--fix` | off | Apply safe auto-fixes (create directories, update config) |

## Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| `critical` | Integration is broken or will produce wrong results | Must fix before using score/refine |
| `warning` | Non-breaking issue that degrades quality or uses deprecated features | Should fix soon |
| `info` | Minor observation or suggestion | Nice to know |

## Workflow

### Step 0: Detect *-apcore Project

1. Check `path` (or CWD) for build config (`pyproject.toml`, `package.json`)
2. Identify the `*-apcore` integration package and version
3. Identify the framework (Flask, Django, NestJS, TipTap, etc.)
4. If not a `*-apcore` project, ask with `AskUserQuestion`

Store: `framework`, `integration_package`, `integration_version`, `language` (python/typescript)

### Step 1: Check Dimension D1 — Dependency Health

**What to check:**

1. **Core apcore SDK installed?**
   - Python: check for `apcore` in installed packages or `pyproject.toml`
   - TypeScript: check `package.json` dependencies for `apcore-js`
   - If missing: `critical` — "apcore SDK not found"

2. **apcore-toolkit installed?**
   - Python: `apcore-toolkit` in dependencies
   - TypeScript: `apcore-toolkit` in dependencies
   - If missing: `warning` — "apcore-toolkit not found; scanner may not work correctly"

3. **Version compatibility**
   - Read the integration package's dependency constraints
   - Check if installed apcore version satisfies the constraint
   - If incompatible: `critical` — "apcore {installed} is not compatible with {integration} (requires {constraint})"

4. **Outdated versions**
   - If apcore-toolkit < 0.2.0: `warning` — "apcore-toolkit {version} is outdated"

### Step 2: Check Dimension D2 — Scanner Configuration

**What to check:**

1. **Scanner entry point exists?**
   - Flask: check for `Apcore(app)` in code or `flask-apcore scan` CLI registration
   - Django: check for `django_apcore` in `INSTALLED_APPS` or `apcore_scan` management command
   - NestJS: check for `ApcoreModule.forRoot()` or `ApToolScannerService` import
   - TipTap: check for `ApcoreModule` or TipTap extension scanning setup
   - If not found: `critical` — "Scanner not configured"

2. **Framework-specific schema backend (Flask only)**
   - Check if Pydantic or marshmallow is installed
   - If neither: `warning` — "No schema backend available (Pydantic or marshmallow). Input schemas will be empty for routes without type hints. Install pydantic for best results."

3. **Include/exclude patterns**
   - Check config for `APCORE_INCLUDE_PATHS` / `APCORE_EXCLUDE_PATHS`
   - If neither is set: `info` — "No include/exclude patterns configured; all endpoints will be scanned"

4. **Endpoint coverage** (best-effort)
   - Try to count routes/endpoints in code vs scanned modules in `.binding.yaml` files
   - If significant gap: `warning` — "Found ~{N} endpoints in code but only {M} in .binding.yaml"

### Step 3: Check Dimension D3 — Output Configuration

**What to check:**

1. **Output format configured?**
   - Check for `--output yaml` in CLI scripts, Makefile, or CI config
   - Check for `APCORE_OUTPUT_FORMAT` setting
   - If using registry-only (no YAML): `warning` — "Using in-memory registry mode. IMPORTANT: The RegistryWriter does NOT pass refined input_schema/output_schema to FunctionModule — any YAML-level schema improvements will be lost at runtime. Switch to YAML output mode for apcore-refinery to be effective, or ensure your code has proper type hints."

2. **Output directory exists?**
   - If YAML output is configured, check the target directory exists
   - If missing: `critical` — "Output directory {path} does not exist"
   - Fixable: create the directory

3. **Binding files present?**
   - If output dir exists, check for `.binding.yaml` files
   - If empty: `warning` — "Output directory exists but contains no .binding.yaml files. Run the scanner."

### Step 4: Check Dimension D4 — Environment & Settings

**What to check:**

1. **Deprecated AI enhancement settings**
   - Check for `APCORE_AI_ENABLED=true` in env, `.env`, or config files
   - Check for `APCORE_SCAN_AI_ENHANCE=true`
   - If found: `warning` — "APCORE_AI_ENABLED is set — the built-in AIEnhancer is deprecated. Use /apcore-refinery:refine instead."
   - Fixable: remove from config

2. **Required settings**
   - Check if `APCORE_ENABLED` is explicitly set (some integrations require it)
   - Check `APCORE_MODULE_DIR` points to a valid directory (if set)
   - If pointing to non-existent path: `critical` — "APCORE_MODULE_DIR={path} does not exist"

3. **Common misconfigurations**
   - `APCORE_DEBUG=true` in production configs: `info`
   - Contradictory settings (AI enhancement enabled but endpoint not configured)

### Step 5: Check Dimension D5 — Annotation Red Flags

Quickly scan the `.binding.yaml` files for **obvious annotation contradictions** that the scanner got wrong. This is NOT a full score — it catches things that are clearly incorrect based on HTTP method semantics and code context.

**What to check:**

1. **GET endpoint with `idempotent: false`**
   - GET requests are inherently idempotent (same request = same response, no side effects)
   - If a GET/HEAD module has `idempotent: false`: `warning` — "{module_id}: GET endpoint has idempotent=false — GET requests are inherently idempotent. Fix: set idempotent=true."
   - Fixable: update the annotation in the binding YAML

2. **`destructive: true` without `requires_approval: true`**
   - Destructive operations (permanent data deletion) should have an approval gate
   - If found: `warning` — "{module_id}: destructive=true but requires_approval=false — permanent data loss without human confirmation. Fix: set requires_approval=true."
   - Fixable: update the annotation

3. **`open_world: true` on in-memory-only operations**
   - For every module with `open_world: true`, resolve the `target` field (e.g., `app:delete_task` → read `app.py`, find `delete_task` function) and read the function body
   - Scan the function for external call indicators: `requests.`, `httpx.`, `urllib`, `aiohttp`, database ORM calls (`session.`, `.query(`, `.execute(`), file I/O (`open(`, `Path(`), subprocess (`subprocess.`, `os.system`)
   - If NONE of these patterns are found and the function only manipulates local variables, dicts, or lists: `warning` — "{module_id}: open_world=true but function body contains no external calls (no HTTP, DB, file I/O, or subprocess). Fix: set open_world=false."
   - This check is important because `open_world` maps to MCP's `openWorldHint` — incorrect values mislead LLM agents about the tool's side effects

4. **`cacheable: true` with `cache_ttl: 0`**
   - Marking something as cacheable but with zero TTL is contradictory
   - If found: `info` — "{module_id}: cacheable=true but cache_ttl=0. Either set a meaningful TTL or set cacheable=false."

5. **`paginated: false` with `pagination_style` set**
   - If `paginated: false` but `pagination_style` has a non-null value: `info` — "{module_id}: paginated=false but pagination_style={value} is set. This is contradictory — remove pagination_style or set paginated=true."

### Step 6: Check Dimension D6 — Known Ecosystem Issues

These are **not** project configuration issues but known bugs/limitations in the apcore ecosystem that affect metadata quality. Reporting them helps users understand why certain refinements may not have the expected effect.

**What to check:**

1. **A2A annotation loss** (if apcore-a2a is in dependencies)
   - `warning` — "apcore-a2a has a known issue: `_build_extensions()` in SkillMapper is defined but never called. All annotation metadata (destructive, readonly, requires_approval) is lost for A2A consumers. Workaround: embed safety signals directly in module descriptions. /apcore-refinery:refine does this automatically for destructive modules."

2. **RegistryWriter schema passthrough** (if using registry output mode)
   - `warning` — "RegistryWriter does not pass `input_schema`/`output_schema` to FunctionModule. FunctionModule always re-generates schemas from function signatures, ignoring any YAML-level refinements. If you need refined schemas at runtime, use YAML output mode with BindingLoader instead."

3. **MCP examples not exposed** (if apcore-mcp is in dependencies)
   - `info` — "apcore-mcp does not include module examples in MCP Tool definitions. Examples are only used by A2A (titles only). However, generating them is still valuable for documentation and future protocol versions."

### Step 7: Output Diagnostic Report

```
apcore-refinery inspect — Integration Health Report

Project: {project_name} ({integration_package} v{version})
Framework: {framework}
Language: {language}

  Dimension                | Critical | Warning | Info
  ─────────────────────────┼──────────┼─────────┼─────
  D1 Dependency Health     |    {n}   |   {n}   |  {n}
  D2 Scanner Config        |    {n}   |   {n}   |  {n}
  D3 Output Config         |    {n}   |   {n}   |  {n}
  D4 Environment           |    {n}   |   {n}   |  {n}
  D5 Annotation Red Flags  |    {n}   |   {n}   |  {n}
  D6 Ecosystem Issues      |    {n}   |   {n}   |  {n}
  ─────────────────────────┼──────────┼─────────┼─────
  TOTAL                    |    {n}   |   {n}   |  {n}

CRITICAL:
  [{dimension}-{seq}] {finding}
    Fix: {suggested fix}

WARNING:
  [{dimension}-{seq}] {finding}
    Fix: {suggested fix or workaround}

INFO:
  [{dimension}-{seq}] {observation}
```

If no critical or warning findings:
```
  All checks passed. Your integration looks healthy.
  Next step: /apcore-refinery:score to assess metadata quality.
```

### Step 8: Auto-Fix (only with --fix)

**Safe auto-fixes (apply directly):**
- Create missing output directories
- Remove deprecated `APCORE_AI_ENABLED` and `APCORE_SCAN_AI_ENHANCE` from config files
- Create missing `__init__.py` files in Python module directories
- Fix `idempotent: false` → `true` on GET/HEAD endpoints in binding YAML
- Fix `requires_approval: false` → `true` on `destructive: true` modules in binding YAML

**Manual-only (show recommendation):**
- Dependency version upgrades
- Scanner configuration changes
- Output mode switches (registry → yaml)
- Ecosystem bug workarounds (A2A annotation loss, RegistryWriter passthrough)

Apply safe fixes, then re-run the report to verify.
