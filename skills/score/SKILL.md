---
name: score
description: >
  Assess and score metadata quality of *-apcore project modules (.binding.yaml files)
  with a deterministic 0-100 rubric across 7 dimensions. Use this skill whenever the
  user asks about metadata quality, wants to know which modules are worst, needs a
  quality gate (pass/fail threshold) for CI/CD, or wants to re-score after running
  refine to verify improvements. Covers flask-apcore, django-apcore, nestjs-apcore,
  tiptap-apcore. Detects silent scanner degradation, reports protocol-specific impact
  (MCP/A2A/CLI), and identifies safety gaps. Read-only — never modifies files.
---

# Apcore Refinery — Score

Deterministic metadata quality assessment for `*-apcore` project modules — not just "what's missing" but "why it's missing" and "which protocol consumers are hurt most."

## Iron Law

**NEVER modify files. Score is read-only.** This skill only reads `.binding.yaml` files, computes scores, and reports results. If you find yourself about to write or edit a file, stop — that's `refine`'s job.

## Why This Skill Exists (Not Just "Count Empty Fields")

A generic quality check misses critical context:
- An empty `input_schema` doesn't mean "no parameters" — it usually means the **scanner silently failed** to extract the schema (Flask: missing Pydantic model, NestJS: @ApTool without inputSchema)
- Missing `x-when-to-use` hurts **MCP most** (appended to Tool description), but is irrelevant to CLI
- Missing `examples` hurts **A2A most** (only titles preserved), but MCP doesn't use them at all
- Missing `tags` only matters for **A2A** skill filtering

This skill encodes which gaps matter to which protocol, and detects scanner degradation patterns that explain WHY quality is low.

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

Store `framework` (flask/django/nestjs/tiptap/other) and `integration_version`.

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

### Step 3: Detect Silent Degradation

After scoring, check each module for scanner degradation patterns (see quality model "Silent Degradation Detection"). These explain WHY a module scores low:

| Pattern | Diagnosis | Framework Context |
|---------|-----------|-------------------|
| `input_schema: {type: object, properties: {}}` on a non-GET endpoint | Schema extraction failed silently | Flask: missing Pydantic/marshmallow model. NestJS: @ApTool without inputSchema. Django: $ref resolution failure. |
| `description` == `documentation` (identical text) | Documentation is just a copy of description | All frameworks: scanner found no distinct docstring body |
| `additionalProperties: true` on input_schema | Permissive fallback from failed extraction | TipTap: unknown extension command. NestJS: DTO extraction failure. |
| All annotations at defaults on a DELETE/PUT/POST endpoint | Annotation inference didn't differentiate | Scanner may not have HTTP method context |
| `destructive: true` but `requires_approval: false` | Safety gap — destructive without approval gate | All frameworks: requires manual override |

Count the degradation findings as a separate diagnostic section.

### Step 4: Protocol Impact Analysis

Group the missing fields by which downstream protocol they hurt most:

**MCP impact** (what LLM agents lose):
- Missing `x-when-to-use` → LLM has no guidance on when to select this tool
- Missing `x-llm-description` on properties → LLM has weaker parameter understanding
- Missing `annotations` → MCP ToolAnnotations hints are all default (misleading)
- `destructive: true` without `requires_approval: true` → dangerous operation without safety gate

**A2A impact** (what agent discovery loses):
- Missing `examples` → AgentSkill has no example titles for discovery
- Missing `tags` → no skill filtering capability
- `description` lacks safety signal → A2A bridge drops all annotations (known bug), so description is the ONLY safety indicator

**CLI impact** (what developers lose):
- Missing `input_schema` property descriptions → --flag help text is empty
- Missing `x-llm-description` → CLI falls back to basic description for help text

### Step 5: Aggregate and Report

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
  ...

  AVERAGE: {avg}/100

  Top Issues:
  1. {N}/{total} modules have placeholder descriptions
  2. {N}/{total} modules have no parameter descriptions in input_schema
  ...

Scanner Degradation Warnings:
  {degradation findings, if any}

Protocol Impact:
  MCP:  {N} modules missing x-when-to-use (LLM tool selection impaired)
        {N} modules have default annotations (safety hints absent)
        {N} modules with destructive=true but requires_approval=false (safety gap)
  A2A:  {N} modules have no examples (skill discovery crippled)
        {N} modules have no tags (filtering unavailable)
        {N} destructive modules lack safety signal in description (A2A drops annotations)
  CLI:  {N} modules have no input_schema descriptions (--flag help empty)

  Tip: Run /apcore-refinery:refine to improve low-scoring modules.
```

#### JSON Format

```json
{
  "project": "{project_name}",
  "framework": "{framework}",
  "date": "{ISO date}",
  "modules": [...],
  "average": 52,
  "top_issues": [...],
  "degradation_warnings": [...],
  "protocol_impact": {
    "mcp": {"missing_intent": N, "default_annotations": N, "safety_gaps": N},
    "a2a": {"missing_examples": N, "missing_tags": N, "missing_safety_signal": N},
    "cli": {"missing_param_descriptions": N}
  }
}
```

### Step 6: Quality Gate

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

If there are degradation warnings, always show them even if the gate passes — they indicate scanner problems that scoring alone doesn't capture.
