---
name: inspect
description: >
  Inspect the health of a *-apcore project integration. Checks dependency versions,
  scanner configuration, output setup, and environment settings. Reports issues with
  severity levels (critical/warning/info) and optional auto-fix for safe remediations.
---

# Apcore Refinery — Inspect

Check whether a `*-apcore` integration is set up correctly before worrying about metadata quality.

## Iron Law

**Diagnose before prescribing.** Inspect finds problems in the integration setup itself — dependency mismatches, missing config, uncovered endpoints. These structural issues should be fixed before spending time on metadata refinement (which would just get overwritten by the next scan anyway).

## When to Use

- First time setting up a `*-apcore` integration — sanity check
- After upgrading apcore or apcore-toolkit — verify compatibility
- When `score` shows unexpectedly bad results — the issue might be config, not metadata
- Periodic health check for an existing integration

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
3. Identify the framework (Flask, Django, NestJS, etc.)
4. If not a `*-apcore` project, ask with `AskUserQuestion`

Store: `framework`, `integration_package`, `integration_version`, `language` (python/typescript)

### Step 1: Check Dimension D1 — Dependency Health

**What to check:**

1. **Core apcore SDK installed?**
   - Python: check for `apcore` in installed packages (`pip show apcore` or parse `pyproject.toml`)
   - TypeScript: check `package.json` dependencies for `apcore-js`
   - If missing: `critical` — "apcore SDK not found"

2. **apcore-toolkit installed?**
   - Python: `apcore-toolkit` in dependencies
   - TypeScript: `apcore-toolkit` in dependencies
   - If missing: `warning` — "apcore-toolkit not found; scanner may not work correctly"

3. **Version compatibility**
   - Read the integration package's dependency constraints (from its `pyproject.toml` or `package.json`)
   - Check if installed apcore version satisfies the constraint
   - If incompatible: `critical` — "apcore {installed} is not compatible with {integration} (requires {constraint})"

4. **Outdated versions**
   - If apcore-toolkit < 0.2.0: `warning` — "apcore-toolkit {version} is outdated"
   - If the integration itself appears to have a newer version available (check changelog or git tags if accessible): `info`

### Step 2: Check Dimension D2 — Scanner Configuration

**What to check:**

1. **Scanner entry point exists?**
   - Flask: check for `flask-apcore scan` CLI command registration, or `Apcore(app)` in code
   - Django: check for `apcore_scan` management command, or `django_apcore` in `INSTALLED_APPS`
   - NestJS: check for `ApcoreModule.forRoot()` or `ApToolScannerService` import
   - If not found: `critical` — "Scanner not configured"

2. **Include/exclude patterns**
   - Check config files for `APCORE_INCLUDE_PATHS` / `APCORE_EXCLUDE_PATHS` or equivalent
   - If neither is set: `info` — "No include/exclude patterns configured; all endpoints will be scanned"

3. **Endpoint coverage** (best-effort)
   - Python/Flask: try to count routes from `app.url_map` references in code
   - Python/Django: try to count URL patterns from `urls.py`
   - Compare to number of scanned modules in `.binding.yaml` files
   - If there's a significant gap: `warning` — "Found ~{N} endpoints in code but only {M} in .binding.yaml"

### Step 3: Check Dimension D3 — Output Configuration

**What to check:**

1. **Output format configured?**
   - Check for `--output yaml` in CLI scripts, Makefile, or CI config
   - Check for `APCORE_OUTPUT_FORMAT` setting
   - If using registry-only (no YAML): `info` — "Using in-memory registry; .binding.yaml not generated. Score/refine require YAML output."

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
   - Check for `APCORE_DEBUG=true` in production-like configs: `info` — "Debug mode is enabled"
   - Check for contradictory settings (e.g., AI enhancement enabled but endpoint not configured)

### Step 5: Output Diagnostic Report

```
apcore-refinery inspect — Integration Health Report

Project: {project_name} ({integration_package} v{version})
Framework: {framework}
Language: {language}

  Dimension              | Critical | Warning | Info
  ───────────────────────┼──────────┼─────────┼─────
  D1 Dependency Health   |    {n}   |   {n}   |  {n}
  D2 Scanner Config      |    {n}   |   {n}   |  {n}
  D3 Output Config       |    {n}   |   {n}   |  {n}
  D4 Environment         |    {n}   |   {n}   |  {n}
  ───────────────────────┼──────────┼─────────┼─────
  TOTAL                  |    {n}   |   {n}   |  {n}

CRITICAL:
  [{dimension}-{seq}] {finding}
    Fix: {suggested fix}

WARNING:
  [{dimension}-{seq}] {finding}
    Fix: {suggested fix}

INFO:
  [{dimension}-{seq}] {observation}
```

If no critical or warning findings:
```
  All checks passed. Your integration looks healthy.
  Next step: /apcore-refinery:score to assess metadata quality.
```

### Step 6: Auto-Fix (only with --fix)

Classify each finding as fixable or manual-only:

**Safe auto-fixes (apply directly):**
- Create missing output directories
- Remove deprecated `APCORE_AI_ENABLED` and `APCORE_SCAN_AI_ENHANCE` from config files
- Create missing `__init__.py` files in Python module directories

**Manual-only (show recommendation):**
- Dependency version upgrades (might break things)
- Scanner configuration changes (project-specific)
- Adding `APCORE_ENABLED=true` to environment (deployment-specific)

Apply safe fixes, then show what was changed:

```
Auto-fixes applied:
  [D3-001] Created directory: ./modules/
  [D4-001] Removed APCORE_AI_ENABLED from .env

Manual action needed:
  [D1-001] Upgrade apcore-toolkit: pip install --upgrade apcore-toolkit
  [D2-001] Configure scanner include paths in settings

Re-running inspect to verify...
{re-run the report}
```
