# Quality Gate Report: flask-apcore Demo Project

- **Date:** 2026-03-17
- **Project:** flask-apcore/examples/demo
- **Threshold:** 60/100
- **Average Score:** 39/100
- **Result: FAIL**

---

## Scoring Methodology

Each `.binding.yaml` file was scored on seven dimensions:

| Dimension | Max Points | What is assessed |
|---|---|---|
| Description quality | 20 | Is the description meaningful, specific, and distinct from documentation? |
| Documentation quality | 15 | Does documentation add context beyond the description (usage notes, examples, caveats)? |
| Tags | 10 | Are tags populated with useful categorization values? |
| Schema completeness | 20 | Are input/output schemas well-defined with types, titles, field descriptions, constraints? |
| Annotation accuracy | 15 | Are behavioral annotations logically correct for the operation type? |
| Metadata richness | 10 | Does metadata go beyond the bare minimum (e.g., author, category, stability)? |
| Semantic precision | 10 | Are individual fields described with human-readable descriptions, examples, or format hints? |

---

## Per-Module Scores

### 1. list_tasks.get.binding.yaml -- Score: 38/100

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 8/20 | Generic one-liner "List all tasks." -- lacks detail on filtering, sort order, or return behavior. |
| Documentation quality | 3/15 | Identical to description; adds no value. |
| Tags | 0/10 | Empty array. |
| Schema completeness | 13/20 | Output schema defines item properties with types and titles. Input schema is empty (no query params for filtering/pagination). |
| Annotation accuracy | 10/15 | `readonly: true` and `cacheable: true` are correct. `idempotent: false` is incorrect for a GET endpoint (GETs are inherently idempotent). `paginated: false` may be a design issue for a list endpoint. |
| Metadata richness | 2/10 | Only `source: native` is present. |
| Semantic precision | 2/10 | Fields have titles but no descriptions or examples. |

### 2. create_task.post.binding.yaml -- Score: 43/100

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 8/20 | "Create a new task." -- does not explain required vs optional fields or what is returned. |
| Documentation quality | 3/15 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Schema completeness | 15/20 | Input schema includes defaults and required markers. Output schema is well-typed. However, no field-level descriptions or string length constraints. |
| Annotation accuracy | 12/15 | Annotations are reasonable. `idempotent: false` is correct for POST. |
| Metadata richness | 2/10 | Only `source: native`. |
| Semantic precision | 3/10 | Titles present but no descriptions, examples, or format/pattern constraints on fields. |

### 3. get_task.get.binding.yaml -- Score: 38/100

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 8/20 | "Get a task by its ID." -- minimal but slightly more specific than others. |
| Documentation quality | 3/15 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Schema completeness | 13/20 | Input has task_id typed as integer with required. Output schema is complete. However, task_id in input lacks a title field. |
| Annotation accuracy | 10/15 | `readonly: true` and `cacheable: true` correct. `idempotent: false` is wrong for a GET. |
| Metadata richness | 2/10 | Only `source: native`. |
| Semantic precision | 2/10 | Input task_id has no title or description. Output fields have titles only. |

### 4. update_task.put.binding.yaml -- Score: 45/100

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 8/20 | "Update an existing task." -- does not specify partial vs full update semantics. |
| Documentation quality | 3/15 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Schema completeness | 16/20 | Best of the group. Uses `anyOf` with null types for optional nullable fields, has defaults. Output schema is complete. |
| Annotation accuracy | 13/15 | `idempotent: true` is correct for PUT. `destructive: false` is reasonable. |
| Metadata richness | 2/10 | Only `source: native`. |
| Semantic precision | 3/10 | Titles present, nullable semantics modeled, but no field descriptions. |

### 5. delete_task.delete.binding.yaml -- Score: 31/100

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 9/20 | "Delete a task permanently." -- slightly better; the word "permanently" adds semantic value. |
| Documentation quality | 3/15 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Schema completeness | 5/20 | Input is minimal (task_id only). Output is a bare `type: object` with no properties -- does not indicate what (if anything) is returned. |
| Annotation accuracy | 11/15 | `destructive: true` is correct. `idempotent: false` is debatable (deleting an already-deleted resource could be considered idempotent). `requires_approval: false` is risky for a destructive operation. |
| Metadata richness | 2/10 | Only `source: native`. |
| Semantic precision | 1/10 | Input task_id has no title or description. Output schema conveys nothing. |

---

## Summary

| Module | Score |
|---|---|
| list_tasks.get | 38 |
| create_task.post | 43 |
| get_task.get | 38 |
| update_task.put | 45 |
| delete_task.delete | 31 |
| **Average** | **39** |

---

## Common Issues

1. **Description and documentation are always identical.** Documentation should expand on the description with usage guidance, error conditions, or behavioral notes.
2. **Tags are universally empty.** Tags like `tasks`, `crud`, `read`, `write`, `destructive` would improve discoverability and categorization.
3. **No field-level descriptions.** Schema properties have `title` but never `description`, leaving consumers without context on semantics, constraints, or expected formats.
4. **Annotation inaccuracies.** GET endpoints are marked `idempotent: false`, which contradicts HTTP semantics. The destructive DELETE does not require approval.
5. **Metadata is minimal.** Only `source: native` is present. Fields like `author`, `category`, `stability`, `since_version` are absent.
6. **delete_task output schema is empty.** Consumers cannot determine what a successful deletion returns.

---

## Verdict

**FAIL** -- Average score of 39/100 does not meet the threshold of 60/100.

The bindings are structurally valid and auto-generated scaffolding is in place, but they lack the descriptive richness, semantic annotations, and documentation depth needed to meet a quality bar of 60. Targeted enrichment of descriptions, documentation, tags, field descriptions, and annotation corrections would be needed to pass.
