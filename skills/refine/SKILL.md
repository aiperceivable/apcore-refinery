---
name: refine
description: >
  Refine and improve *-apcore module metadata by reading source code and generating
  high-quality descriptions, input_schema property descriptions, x-llm-description,
  documentation, behavioral annotations, examples, tags, and MCP/A2A/CLI intent
  metadata (x-when-to-use, x-when-not-to-use, x-common-mistakes, x-workflow-hints).
  Use this skill when the user wants to improve, enhance, enrich, or fix module
  metadata quality, make metadata more useful for LLM agents, or generate better
  descriptions from source code. Covers flask-apcore, django-apcore, nestjs-apcore,
  tiptap-apcore. Protocol-aware — generates fields optimized for MCP, A2A, and CLI
  simultaneously. Shows diff for human review before writing. No external API needed.
---

# Apcore Refinery — Refine

Read source code. Understand intent. Generate protocol-aware metadata. Show diff. Apply on approval.

## Iron Law

**ALWAYS show diff before writing. Never silently overwrite.** The user must see every proposed change and explicitly approve before any file is modified. An incorrect `destructive: false` annotation could cause an LLM agent to execute a dangerous operation without approval — accuracy matters more than coverage.

## Why This Skill Exists (Not Just "Improve the YAML")

A generic "improve metadata" approach misses critical protocol-specific behavior:

- The MCP bridge **appends** `x-when-to-use` to the Tool description — writing it wrong means the LLM sees garbled text
- The A2A bridge **discards** example inputs/outputs, keeping only titles — titles must be self-descriptive
- The A2A bridge has a **known bug** where annotations are completely lost — safety info must be embedded in descriptions for destructive modules
- The CLI bridge **truncates** `x-llm-description` to 200 chars and promotes `_file`-suffixed params to file paths
- Each framework scanner has characteristic **silent degradation** patterns — empty schemas don't mean "no parameters"

This skill encodes all of this domain knowledge so refinements produce metadata that works correctly across MCP, A2A, and CLI consumers simultaneously.

## Command Format

```
/apcore-refinery:refine [path] [--apply] [--modules id1,id2,...] [--threshold N]
```

| Flag | Default | Description |
|------|---------|-------------|
| `path` | CWD | Directory containing `.binding.yaml` files |
| `--apply` | off | Skip diff review and write changes directly |
| `--modules` | — | Comma-separated module IDs to refine (skip scoring, refine only these) |
| `--threshold` | `70` | Only refine modules scoring below this threshold |

## Workflow

### Step 0: Detect *-apcore Project and Framework

Check `pyproject.toml` / `package.json` for `*-apcore` dependency. Store:
- `framework` (flask/django/nestjs/tiptap/other)
- `language` (python/typescript)

The framework matters because each scanner has different quality gaps. See the framework-specific patterns in the quality model.

### Step 1: Identify Modules to Refine

**If `--modules` specified:** find the `.binding.yaml` files containing those module IDs. Skip scoring.

**Otherwise:** run the scoring logic from the quality model to identify modules below `--threshold`.

@../shared/quality-model.md

Display what will be refined:

```
Modules to refine ({count} below threshold {threshold}):

  Module ID                    | Current Score
  users.create_user.post       |   32
  orders.cancel_order.delete   |   45
  payments.process_payment.post|   28

Modules skipped ({count} at or above threshold):
  users.get_user.get           |   78
```

If all modules are above threshold, say so and suggest `--threshold 90` for further polish. Then stop.

### Step 2: Detect Silent Degradation

Before reading source code, check each module for signs of scanner degradation (see quality model "Silent Degradation Detection"):

- `input_schema` is empty `{type: object, properties: {}}` — likely schema extraction failure, NOT "no parameters"
- `description` == `documentation` — documentation is just a copy
- `additionalProperties: true` — permissive fallback from failed extraction
- Empty annotations despite clear HTTP method context

Report any findings:
```
Degradation detected:
  create_task.post: Empty input_schema — likely missing Pydantic model (Flask pattern)
  delete_task.delete: documentation duplicates description
```

This context helps the user understand WHY the metadata is bad, not just that it scores low.

### Step 3: Resolve Source Code

For each module to refine, parse the `target` field:

**Python targets** (`module.path:qualname`):
1. Convert module path to file path: `module.path` → `module/path.py`
2. Search in `src/`, project root, and common Python source directories
3. Read the file and locate the function/class by `qualname`

**TypeScript targets** (`file/path:exportName`):
1. Look for the file directly, or try with `.ts`/`.js` extensions
2. Read the file and locate the named export

**If source cannot be found:** proceed with metadata-only refinement (less accurate) and note the warning.

**Context gathering:** For each function, also read:
- Imports (to understand framework and dependencies)
- Related functions called by this function (one level deep, same file)
- Pydantic/marshmallow/DTO model definitions used as parameters
- Type hints and return type

### Step 4: Generate Protocol-Aware Refinements

For each module, read the source code alongside the current YAML. Generate improvements following the protocol-specific formatting rules in the quality model.

@../shared/quality-model.md

**Field-by-field generation guide:**

#### 4.1 description

Write a clear, action-oriented sentence (< 200 chars) that tells an LLM agent exactly what this module does. Start with a verb.

For **destructive** or **requires_approval** modules, embed a safety signal directly in the description — because the A2A bridge loses all annotation metadata:
- `"Permanently delete a task by its ID (destructive, irreversible)"`
- `"Transfer funds between accounts (requires human approval)"`

This safety embedding is NOT redundant — it's the only way A2A agents learn about dangerous operations.

#### 4.2 input_schema property descriptions + x-llm-description

For every property in `input_schema.properties`:

1. Add a `description` field explaining the parameter's business meaning based on code usage
2. Add `x-llm-description` with an **instruction-style** description for LLM callers — this gets promoted in OpenAI strict mode and used as CLI --flag help text (max 200 chars)
3. Add `enum` constraints if the code checks against a fixed set of values
4. Add `x-cli-file: true` if the parameter accepts a file path

Example:
```yaml
task_id:
  type: integer
  description: The unique identifier of the task to operate on
  x-llm-description: "Integer ID of the target task. Use list_tasks.get to discover valid IDs."
```

#### 4.3 documentation

Write 2-5 sentences covering: what the module does, side effects (DB writes, emails, external API calls), permissions needed, error conditions, and return value format. Must exceed 50 characters. This becomes an MCP Resource at `docs://{module_id}`.

#### 4.4 annotations

Infer from source code using the annotation inference guide in the quality model. Be conservative — a wrong annotation is worse than a default one.

Key safety rule: if `destructive: true`, strongly consider also setting `requires_approval: true` (the delete_task pattern — permanent data loss warrants human confirmation).

#### 4.5 examples

Generate 1-2 realistic examples with `title`, `inputs`, and `output`.

**Critical A2A consideration:** The A2A bridge extracts **only the title** (max 10), discarding inputs and outputs. Write titles that are self-descriptive:
- Bad: `"Basic example"`, `"Test"`, `"Example 1"`
- Good: `"Create admin user with full permissions"`, `"Delete completed task by ID 42"`

Use plausible values from the code's domain (e.g., task titles from the in-memory store), not `"test"` or `"foo"`.

#### 4.6 tags

Derive from:
- Module's business domain (e.g., `tasks`, `users`, `orders`)
- Operation type (e.g., `create`, `delete`, `list`, `update`)
- Framework blueprint/module namespace if available
- Keep to 3-5 tags maximum

#### 4.7 metadata: x-when-to-use, x-when-not-to-use, x-common-mistakes

These are appended to the MCP Tool description as labeled sections. The LLM will see them directly after the base description.

- **x-when-to-use**: Write as an instruction to the LLM. Start with "Use when..." or "When the user wants to...". Do NOT repeat the description.
- **x-when-not-to-use**: Reference **specific alternative modules by name** if they exist in the same project. E.g., "For updating existing tasks, use update_task.put instead."
- **x-common-mistakes**: Describe **caller errors** (things the LLM might get wrong), not user errors. E.g., "Omitting the required title field" or "Passing an ID to create (IDs are auto-assigned)."

#### 4.8 metadata: x-workflow-hints (optional)

If the module is part of a natural workflow (e.g., list → get → update → delete), add `x-workflow-hints` describing the typical sequence. The MCP bridge appends this as "Workflow Hints: ..." in the description.

**Critical rules:**
- **Never rename parameters.** The `input_schema` property names must exactly match the function's parameter names.
- **Accuracy over coverage.** If you can't confidently determine an annotation from the code, leave it at default.
- **Use the code, not imagination.** Every description, example, and annotation must be grounded in source code behavior.

**Parallelization:** If more than 5 modules need refinement, use sub-agents (one per module). Each sub-agent receives: the module's YAML, the source code, and the relevant quality model sections inline (not file references).

### Step 5: Show Diff

For each refined module, display a clear before/after comparison grouped by dimension:

```
── {module_id} ({old_score} → {new_score}) ──

  description:
  - "{old}"
  + "{new}"

  input_schema.properties.{param}:
  + description: "{generated}"
  + x-llm-description: "{instruction-style for LLM/CLI}"

  annotations:
  + destructive: true
  + requires_approval: true

  + documentation: |
  +   {multi-sentence documentation}

  + examples:
  +   - title: "{self-descriptive title for A2A}"
  +     inputs: {realistic data}
  +     output: {matching output}

  + metadata:
  +   x-when-to-use: "{LLM instruction, no repeat of description}"
  +   x-when-not-to-use: "{references specific alternative modules}"
  +   x-common-mistakes: "{caller errors the LLM might make}"

  + tags: [{domain tags}]
```

After all diffs, show summary:
```
Summary: {count} modules refined
  Average score: {old_avg} → {new_avg} (+{delta})

Protocol impact:
  MCP: {count} modules now have x-when-to-use guidance for LLM tool selection
  A2A: {count} modules now have self-descriptive example titles
  CLI: {count} modules now have x-llm-description for --flag help text
```

### Step 6: Apply Changes

**If `--apply`:** Write all changes immediately, then verify.

**Otherwise:** Ask:
```
Apply changes?
  [A] Apply all ({count} modules)
  [S] Select individual modules to apply
  [N] Cancel — no changes written
```

After writing, re-score and display before/after comparison with file paths.
