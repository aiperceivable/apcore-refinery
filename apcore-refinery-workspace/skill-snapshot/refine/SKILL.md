---
name: refine
description: >
  Refine *-apcore module metadata by reading source code and generating high-quality
  descriptions, schema annotations, behavioral flags, examples, tags, and MCP/A2A/CLI
  intent metadata. Shows a diff for human review before writing changes.
  Claude Code itself performs the refinement — no external LLM API calls needed.
---

# Apcore Refinery — Refine

Read source code. Understand intent. Generate protocol-aware metadata. Show diff. Apply on approval.

## Iron Law

**ALWAYS show diff before writing. Never silently overwrite.** The user must see every proposed change and explicitly approve before any file is modified. An incorrect `destructive: false` annotation could cause an LLM agent to execute a dangerous operation without approval — accuracy matters more than coverage.

## When to Use

- After `score` reveals low-quality modules
- When onboarding legacy code to apcore and the scanner output is bare-bones
- When preparing modules for MCP/A2A exposure and the metadata needs to be LLM-friendly
- After code changes that affect module behavior (annotations may need updating)

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

### Step 0: Detect *-apcore Project

Same detection as score: check `pyproject.toml` / `package.json` for `*-apcore` dependency. Store `framework` type.

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

If all modules are above threshold:
```
All {count} modules score >= {threshold}. Nothing to refine.
Tip: Use --threshold 90 to target modules for further polish.
```
Then stop.

### Step 2: Resolve Source Code

For each module to refine, parse the `target` field to locate source code:

**Python targets** (format: `module.path:qualname`):
1. Convert module path to file path: `module.path` → `module/path.py`
2. Search in `src/`, project root, and common Python source directories
3. Read the file and locate the function/class by `qualname`

**TypeScript targets** (format: `file/path:exportName`):
1. Look for the file directly, or try with `.ts`/`.js` extensions
2. Read the file and locate the named export

**If source cannot be found:**
- Log a warning: `"Cannot resolve source for {module_id} (target: {target}). Will refine from metadata only."`
- Proceed with metadata-only refinement (less accurate but still useful for descriptions and tags)

**Context gathering:** For each resolved function, also read:
- Imports at the top of the file (to understand dependencies and frameworks used)
- Related functions called by this function (one level deep, if in the same file)
- The function's type hints and return type

### Step 3: Generate Refinements

For each module, read the source code alongside the current `.binding.yaml` content. Generate improvements for every dimension that scores below its maximum.

Refer to the protocol requirements matrix in the quality model to understand what each field is used for downstream:

@../shared/quality-model.md

**What to generate for each module:**

1. **description** — Write a clear, action-oriented sentence (< 200 chars) that tells an LLM agent exactly what this module does. Start with a verb. Bad: `"POST /api/users"`. Good: `"Create a new user account with the specified name and role"`.

2. **input_schema property descriptions** — For every property in `input_schema.properties`, add a `description` field explaining what the parameter means in business terms. Base this on how the parameter is used in the code, not just its name or type. Also add `enum` constraints if the code checks against a fixed set of values.

3. **documentation** — Write 2-5 sentences covering: what the module does, what side effects it has (DB writes, emails sent, external API calls), what permissions are needed, and what errors it can return.

4. **annotations** — Infer from the source code using the annotation inference guide in the quality model. Be conservative: if unsure whether something is destructive, leave it as default rather than marking `destructive: false` incorrectly.

5. **examples** — Generate 1-2 realistic examples with `title`, `inputs`, and `output` based on the code's actual data model. Use plausible values, not `"test"` or `"foo"`.

6. **tags** — Derive from the module's domain (e.g., `users`, `orders`, `payments`), the HTTP method context (e.g., `create`, `delete`, `list`), and any business domain visible in the code.

7. **metadata.x-when-to-use** — One sentence: when should an LLM agent choose this tool?

8. **metadata.x-when-not-to-use** — One sentence: when should an LLM agent NOT choose this tool? Reference alternative modules if they exist in the same project.

9. **metadata.x-common-mistakes** — One sentence: what do callers commonly get wrong?

**Critical rules:**
- **Never rename parameters.** The `input_schema` property names must exactly match the function's parameter names. Renaming breaks the call chain.
- **Accuracy over coverage.** If you can't confidently determine an annotation from the code, leave it at its default. A wrong annotation is worse than a missing one.
- **Use the code, not imagination.** Every description, example, and annotation must be grounded in what the source code actually does.

**Parallelization:** If more than 5 modules need refinement, use sub-agents (one per module) to process them in parallel. Each sub-agent receives:
- The module's current YAML content
- The resolved source code
- The quality model content (copy the relevant sections inline — don't reference file paths)

Sub-agent prompt template:
```
Refine the metadata for this apcore module. Read the source code carefully and generate
improvements for each dimension below.

Current YAML:
{yaml_content}

Source code:
{source_code}

Generate a complete replacement YAML with all improvements applied. Keep the module_id,
target, and version unchanged. Keep all input_schema property NAMES unchanged (only add
descriptions and constraints). Follow these rules:
{quality_model_excerpt}

Return ONLY the complete YAML content, no explanation.
```

### Step 4: Show Diff

For each refined module, display a clear before/after comparison. Group changes by dimension so the user can quickly see what changed.

```
── {module_id} ({old_score} → {new_score}) ──

  description:
  - "{old_description}"
  + "{new_description}"

  input_schema.properties.{param_name}:
  + description: "{generated description}"

  input_schema.properties.{param_name}:
  + description: "{generated description}"
  + enum: [{values}]

  annotations:
  + destructive: true
  + requires_approval: true

  + documentation: |
  +   {generated documentation}

  + examples:
  +   - title: "{example title}"
  +     inputs: {example inputs}
  +     output: {example output}

  + metadata:
  +   x-when-to-use: "{guidance}"
  +   x-when-not-to-use: "{guidance}"
  +   x-common-mistakes: "{guidance}"

  + tags: [{tags}]
```

After showing all diffs, display a summary:
```
Summary: {count} modules refined
  Average score: {old_avg} → {new_avg} (+{delta})
```

### Step 5: Apply Changes

**If `--apply` was specified:** Write all changes immediately, then proceed to verification.

**Otherwise:** Ask the user:

```
Apply changes?
  [A] Apply all ({count} modules)
  [S] Select individual modules to apply
  [N] Cancel — no changes written
```

- **Apply all**: Write every refined module's YAML back to disk
- **Select**: List modules with their score improvements, let user pick which to apply
- **Cancel**: Exit without any file modifications

**After writing:**

1. Re-read the written files and re-score them to verify improvements
2. Display before/after comparison:

```
Quality improvement:

  Module ID                    | Before | After | Delta
  users.create_user.post       |   32   |   87  |  +55
  orders.cancel_order.delete   |   45   |   82  |  +37
  payments.process_payment.post|   28   |   90  |  +62

  Average: {old_avg} → {new_avg} (+{delta})

Changes written to:
  modules/users_create_user_post.binding.yaml
  modules/orders_cancel_order_delete.binding.yaml
  modules/payments_process_payment_post.binding.yaml
```
