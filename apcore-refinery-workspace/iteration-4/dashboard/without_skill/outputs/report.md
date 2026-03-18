# apcore-refinery score -- Quality Report

**Project:** flask-apcore demo (flask-apcore v0.3.0)
**Modules:** 5 YAML bindings + 1 in-memory @module (task_stats.v1, no binding file)
**Date:** 2026-03-17

---

## Score Table

| Module ID              | Total | Desc | Schema | Docs | Annot | Ex  | Tags | Intent |
|------------------------|------:|-----:|-------:|-----:|------:|----:|-----:|-------:|
| create_task.post       |    25 |   25 |      0 |    0 |     0 |   0 |    0 |      0 |
| delete_task.delete     |    40 |   25 |      0 |    0 |    15 |   0 |    0 |      0 |
| get_task.get           |    40 |   25 |      0 |    0 |    15 |   0 |    0 |      0 |
| list_tasks.get         |    65 |   25 |     25 |    0 |    15 |   0 |    0 |      0 |
| update_task.put        |    40 |   25 |      0 |    0 |    15 |   0 |    0 |      0 |

**AVERAGE: 42/100**

---

## Top Issues

1. **5/5 modules have no documentation** -- `documentation` field is either missing or identical to `description` and under 50 characters. No module has a distinct, detailed docstring.
2. **4/5 modules have no input_schema property descriptions** -- Properties like `task_id`, `title`, `description`, `done` all lack a `description` field. Only `list_tasks.get` scores full marks here because it has no input properties at all (vacuously complete).
3. **5/5 modules have no examples** -- No module provides an `examples` list with title and inputs.
4. **5/5 modules have no tags** -- Every module has `tags: []`.
5. **5/5 modules have no intent metadata** -- No module defines `metadata.x-when-to-use`.
6. **1/5 modules has all-default annotations** -- `create_task.post` has all annotations at their default values (the scanner did not differentiate it from a read-only endpoint).

---

## Scanner Degradation Warnings

| Module | Pattern | Diagnosis |
|--------|---------|-----------|
| create_task.post | All annotations at defaults on a POST endpoint | Annotation inference did not set `idempotent: false` explicitly vs. leaving everything default. The scanner recognized it as non-readonly but did not mark any annotation as non-default. |
| delete_task.delete | `destructive: true` but `requires_approval: false` | Safety gap -- a destructive delete operation has no approval gate. This is a policy decision but worth flagging. |
| All 5 modules | `description` == `documentation` (identical text) | The scanner copied the Flask docstring into both fields. No distinct extended documentation exists. |

---

## Protocol Impact

### MCP (LLM agent tool selection)
- **5/5** modules missing `x-when-to-use` -- LLM has no guidance on when to select each tool
- **4/5** modules missing `x-llm-description` on input properties -- LLM has weaker parameter understanding
- **1/5** modules (`create_task.post`) has all-default annotations -- MCP ToolAnnotations hints are misleading (appears read-only/safe when it is a write operation)
- **1/5** modules (`delete_task.delete`) is `destructive: true` without `requires_approval: true` -- dangerous operation without safety gate

### A2A (agent-to-agent skill discovery)
- **5/5** modules have no examples -- AgentSkill has no example titles for discovery
- **5/5** modules have no tags -- skill filtering unavailable
- **1/1** destructive module (`delete_task.delete`) description says "Delete a task permanently." which does convey danger, but lacks an explicit safety signal like "(destructive, irreversible)" -- A2A bridge drops all annotations (known bug), so description is the ONLY safety indicator

### CLI (developer experience)
- **4/5** modules have no input_schema property descriptions -- `--flag` help text will be empty
- **0/5** modules have `x-llm-description` on properties -- CLI falls back to basic description for help text

---

## In-Memory Module Note

`task_stats.v1` is registered via the `@module` decorator in `app.py` but has **no `.binding.yaml` file**. It exists only in-memory at runtime. To include it in quality scoring, export it to a binding YAML file first (e.g., via `flask-apcore scan`).

---

## Available Commands to Improve

The **apcore-refinery** toolkit provides three commands you can run to assess and improve metadata quality:

### `/apcore-refinery:score [path] [--format table|json] [--fail-under N]`
What you just saw above. Deterministic 0-100 quality scoring per module. Use `--fail-under 70` in CI to enforce a quality gate.

### `/apcore-refinery:refine [path] [--apply] [--modules id1,id2,...] [--threshold N]`
AI-assisted metadata improvement. Reads your source code and binding YAML files, then generates enriched metadata: better descriptions, property descriptions, `x-when-to-use` intent, examples, tags, and corrected annotations. Use `--apply` to write changes, or review the diff first without it. Use `--modules` to target specific modules or `--threshold N` to only refine modules scoring below N.

### `/apcore-refinery:inspect [path] [--fix]`
Structural health check of your `*-apcore` integration: validates binding file syntax, checks target resolution (do the referenced functions exist?), verifies schema consistency, and detects configuration issues. Use `--fix` to auto-repair simple problems.

---

## Recommended Next Steps

1. **Run `/apcore-refinery:refine`** to enrich all 5 modules -- this is the highest-leverage action, likely raising the average from 42 to 75+ by adding documentation, property descriptions, intent metadata, examples, and tags.
2. **Review the `delete_task.delete` safety gap** -- consider setting `requires_approval: true` for the destructive delete endpoint, especially if MCP approval is enabled.
3. **Fix `create_task.post` annotations** -- at minimum, the scanner should have set meaningful annotations for a POST endpoint (not all defaults).
4. **Export `task_stats.v1`** to a binding YAML so it is included in quality tracking.
