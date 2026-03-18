---
name: score
description: >
  Assess metadata quality of *-apcore project modules with a deterministic 0-100
  score per module. Reads .binding.yaml files, applies the quality model rubric,
  and outputs a detailed report. Supports --fail-under for CI/CD quality gates.
  This is a read-only operation — it never modifies files.
---

# Apcore Refinery — Score

Deterministic metadata quality assessment for `*-apcore` project modules.

## Iron Law

**NEVER modify files. Score is read-only.** This skill only reads `.binding.yaml` files, computes scores, and reports results. If you find yourself about to write or edit a file, stop — that's `refine`'s job.

## When to Use

- User wants to know how good their module metadata is
- Before running `refine`, to see which modules need improvement
- In CI/CD, with `--fail-under` to enforce quality gates
- After `refine`, to verify improvements

## Command Format

```
/apcore-refinery:score [path] [--format table|json] [--fail-under N]
```

| Flag | Default | Description |
|------|---------|-------------|
| `path` | CWD | Directory to scan for `.binding.yaml` files |
| `--format` | `table` | Output format: `table` or `json` |
| `--fail-under` | — | If set, report PASS/FAIL based on average score vs threshold |

## Workflow

### Step 0: Detect *-apcore Project

1. Check `path` (or CWD) for `pyproject.toml` or `package.json`
2. Look for `*-apcore` dependency to identify framework type
3. If not a `*-apcore` project, ask user to confirm with `AskUserQuestion`

Store `framework` (flask/django/nestjs/other) and `integration_version`.

### Step 1: Discover Module Definitions

Find all `.binding.yaml` files:

1. Glob `**/*.binding.yaml` in project directory (max depth 3)
2. Also check: `bindings/`, `modules/`, `apcore_modules/`, `output/`
3. If no files found, tell the user:
   ```
   No .binding.yaml files found. Run your framework's scanner first:
     flask-apcore:   flask-apcore scan --output yaml ./modules/
     django-apcore:  python manage.py apcore_scan --output yaml ./modules/
     nestjs-apcore:  (modules are registered in-memory; export with --output yaml)
   ```
   Then stop.

### Step 2: Score Each Module

Read each `.binding.yaml` file and apply the quality model.

@../shared/quality-model.md

For each module in each YAML file, compute:

1. **Description Score (0-25)**: Apply placeholder detection rules. 25 if non-placeholder and >= 10 chars, 0 otherwise.

2. **Schema Description Score (0-25)**: Count `input_schema.properties` that have a `description` field. Score = `25 * (described / total)`. If the schema has no properties (empty or `{}`), score is 25 (vacuously complete — there's nothing to describe).

3. **Documentation Score (0-15)**: 15 if `documentation` field is non-null and longer than 50 characters, 0 otherwise.

4. **Annotations Score (0-15)**: 15 if at least one annotation differs from the all-default state. The defaults are: `readonly=false, destructive=false, idempotent=false, requires_approval=false, open_world=true, streaming=false, cacheable=false, paginated=false`. If annotations are null/missing or all match defaults, score is 0.

5. **Examples Score (0-10)**: 10 if `examples` list contains at least one entry with a `title` and `inputs`, 0 otherwise.

6. **Tags Score (0-5)**: 5 if `tags` is a non-empty list, 0 otherwise.

7. **Intent Metadata Score (0-5)**: 5 if `metadata` dict contains `x-when-to-use` key, 0 otherwise.

**Total = sum of all dimensions (0-100).**

### Step 3: Aggregate and Report

Compute:
- Per-module scores and dimension breakdowns
- Average score across all modules
- Top issues (most common zero-scoring dimensions)

#### Table Format (default)

```
apcore-refinery score — Quality Report

Project: {project_name} ({framework}-apcore v{version})
Modules: {count}
Date: {date}

  Module ID                    | Total | Desc | Schema | Docs | Annot | Ex  | Tags | Intent
  ─────────────────────────────┼───────┼──────┼────────┼──────┼───────┼─────┼──────┼───────
  users.create_user.post       |   32  |   0  |    8   |   0  |   0   |   0 |   0  |   0
  users.get_user.get           |   78  |  25  |   25   |  15  |   0   |   0 |   5  |   0
  users.list_users.get         |   55  |  25  |   25   |   0  |   0   |   0 |   5  |   0
  orders.cancel_order.delete   |   45  |  25  |   12   |   0  |   0   |   0 |   0  |   0
  ...

  AVERAGE: {avg}/100

  Top Issues:
  1. {N}/{total} modules have placeholder descriptions
  2. {N}/{total} modules have no parameter descriptions in input_schema
  3. {N}/{total} modules have default annotations (none explicitly set)
  4. {N}/{total} modules have no examples
  5. {N}/{total} modules missing x-when-to-use metadata

  Tip: Run /apcore-refinery:refine to improve low-scoring modules.
```

#### JSON Format

```json
{
  "project": "{project_name}",
  "framework": "{framework}",
  "date": "{ISO date}",
  "modules": [
    {
      "module_id": "users.create_user.post",
      "total": 32,
      "dimensions": {
        "description": 0,
        "schema_descriptions": 8,
        "documentation": 0,
        "annotations": 0,
        "examples": 0,
        "tags": 0,
        "intent_metadata": 0
      },
      "issues": ["placeholder_description", "missing_documentation", "default_annotations", "no_examples", "no_intent_metadata"]
    }
  ],
  "average": 52,
  "top_issues": [...]
}
```

### Step 4: Quality Gate

If `--fail-under N` is specified:

- If average score >= N:
  ```
  Quality gate PASSED: {avg}/100 >= {N}
  ```
- If average score < N:
  ```
  Quality gate FAILED: {avg}/100 < {N}

  {count} modules below threshold. Run /apcore-refinery:refine to improve.
  ```
