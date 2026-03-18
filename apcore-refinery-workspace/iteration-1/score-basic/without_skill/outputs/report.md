# Module Metadata Quality Report

**Project:** flask-apcore demo
**Source:** `/Users/tercelyi/Workspace/aipartnerup/flask-apcore/examples/demo/apcore_modules/`
**Date:** 2026-03-17
**Modules assessed:** 5

---

## Scoring Methodology

Each module is scored across 8 quality dimensions on a 0-10 scale. The per-module score is the average of all dimension scores.

| Dimension | What it measures |
|---|---|
| **Description quality** | Is the description informative, specific, and distinct from the module ID? |
| **Documentation quality** | Does `documentation` add value beyond `description`? |
| **Tags** | Are meaningful tags provided for categorization and discovery? |
| **Input schema completeness** | Are input properties well-typed, titled, described, and have constraints? |
| **Output schema completeness** | Are output properties well-typed, titled, described, and have constraints? |
| **Annotation accuracy** | Are behavioral annotations (readonly, destructive, idempotent, etc.) correct and complete? |
| **Metadata richness** | Does the metadata section carry useful provenance or context? |
| **Overall consistency** | Is the binding internally consistent and free of contradictions? |

---

## Per-Module Scores

### 1. `list_tasks.get` (list_tasks.get.binding.yaml)

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 4/10 | "List all tasks." is terse and generic. Does not mention filtering, domain context, or what a "task" is. |
| Documentation quality | 1/10 | Identical to description -- adds zero additional information. |
| Tags | 0/10 | Empty array `[]`. No tags for categorization. |
| Input schema completeness | 5/10 | Correctly declares no required input (empty properties object), but there is no documentation about optional query parameters like filtering or pagination. |
| Output schema completeness | 7/10 | Array of Task objects with typed, titled fields. Missing: field-level descriptions, format hints (e.g., max length on strings), and `items.type: object` is present. |
| Annotation accuracy | 7/10 | `readonly: true` and `cacheable: true` are correct. `idempotent: false` is wrong -- a GET that lists resources IS idempotent. `cache_ttl: 0` undermines `cacheable: true`. `paginated: false` with `pagination_style: cursor` is contradictory. |
| Metadata richness | 2/10 | Only `source: native`. No author, category, or domain tags. |
| Overall consistency | 5/10 | The paginated/pagination_style contradiction and cacheable with ttl=0 reduce consistency. |

**Module score: 3.9 / 10**

---

### 2. `create_task.post` (create_task.post.binding.yaml)

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 4/10 | "Create a new task." is minimal. Does not describe what fields are expected or what side effects occur. |
| Documentation quality | 1/10 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Input schema completeness | 7/10 | Has typed properties with titles, defaults for optional fields, and a `required` list. Missing: field-level descriptions, string length constraints, and validation rules. |
| Output schema completeness | 7/10 | Returns a full Task object with all fields typed, titled, and required. Missing: field-level descriptions. |
| Annotation accuracy | 7/10 | `readonly: false`, `destructive: false` are correct for a create operation. `idempotent: false` is correct (POST is not idempotent). `open_world: true` is debatable -- this is a well-defined local operation. `paginated: false` with `pagination_style: cursor` is contradictory. |
| Metadata richness | 2/10 | Only `source: native`. |
| Overall consistency | 6/10 | Minor contradiction on pagination_style when paginated is false. |

**Module score: 4.3 / 10**

---

### 3. `get_task.get` (get_task.get.binding.yaml)

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 5/10 | "Get a task by its ID." is slightly more specific than others but still minimal. |
| Documentation quality | 1/10 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Input schema completeness | 5/10 | Has `task_id` with type and required. Missing: title, description, minimum value constraint. |
| Output schema completeness | 7/10 | Full Task object, typed and titled. Missing: field-level descriptions. |
| Annotation accuracy | 7/10 | `readonly: true`, `cacheable: true` are correct. `idempotent: false` is wrong -- GET by ID IS idempotent. `cache_ttl: 0` contradicts `cacheable: true`. `paginated: false` / `pagination_style: cursor` contradiction. |
| Metadata richness | 2/10 | Only `source: native`. |
| Overall consistency | 5/10 | Same contradictions as list_tasks. |

**Module score: 4.0 / 10**

---

### 4. `update_task.put` (update_task.put.binding.yaml)

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 4/10 | "Update an existing task." is generic. Does not clarify partial vs. full update semantics. |
| Documentation quality | 1/10 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Input schema completeness | 7/10 | Good use of `anyOf` with null types for optional partial-update fields. Has defaults, titles, required list. Missing: field-level descriptions, string constraints. |
| Output schema completeness | 7/10 | Full Task object. Missing: field-level descriptions. |
| Annotation accuracy | 8/10 | `idempotent: true` is correct for PUT. `readonly: false`, `destructive: false` are correct. Same `paginated: false` / `pagination_style: cursor` contradiction. `open_world: true` is debatable. |
| Metadata richness | 2/10 | Only `source: native`. |
| Overall consistency | 6/10 | Minor pagination contradiction. Input schema's partial-update pattern (nullable defaults) is a good sign of intentional design. |

**Module score: 4.4 / 10**

---

### 5. `delete_task.delete` (delete_task.delete.binding.yaml)

| Dimension | Score | Notes |
|---|---|---|
| Description quality | 5/10 | "Delete a task permanently." is slightly better -- the word "permanently" conveys irreversibility. |
| Documentation quality | 1/10 | Identical to description. |
| Tags | 0/10 | Empty array. |
| Input schema completeness | 5/10 | Has `task_id` with type and required. Missing: title, description, minimum value constraint. |
| Output schema completeness | 2/10 | Only `type: object` with no properties defined. Does not indicate what (if anything) is returned -- e.g., a success message, the deleted object, or an empty body. |
| Annotation accuracy | 7/10 | `destructive: true` is correct and important. `readonly: false` correct. `idempotent: false` is debatable -- DELETE is often considered idempotent (deleting an already-deleted resource returns the same outcome). `requires_approval: false` is risky for a destructive operation. Same pagination contradiction. |
| Metadata richness | 2/10 | Only `source: native`. |
| Overall consistency | 5/10 | Sparse output schema plus `destructive: true` without `requires_approval: true` is a safety concern. |

**Module score: 3.4 / 10**

---

## Summary

| Module | Score | Grade |
|---|---|---|
| `list_tasks.get` | 3.9 / 10 | Poor |
| `create_task.post` | 4.3 / 10 | Below Average |
| `get_task.get` | 4.0 / 10 | Poor |
| `update_task.put` | 4.4 / 10 | Below Average |
| `delete_task.delete` | 3.4 / 10 | Poor |
| **Overall average** | **4.0 / 10** | **Poor** |

---

## Top Issues (by impact)

1. **Documentation is always identical to description (all 5 modules).** The `documentation` field should provide richer detail -- usage examples, error conditions, expected behavior, and edge cases. As-is, it is pure duplication and adds no value.

2. **No tags on any module.** Tags like `tasks`, `crud`, `read`, `write`, `destructive` would aid discovery, filtering, and AI-based routing.

3. **Idempotency incorrectly set on read operations.** `list_tasks.get` and `get_task.get` both have `idempotent: false`, but GET requests are inherently idempotent. This could mislead consumers or AI agents about retry safety.

4. **Pagination contradiction on all modules.** Every module sets `paginated: false` but also `pagination_style: cursor`. If pagination is disabled, `pagination_style` should be `null` or omitted entirely.

5. **Cache TTL of 0 undermines cacheable flag.** `list_tasks.get` and `get_task.get` are marked `cacheable: true` but have `cache_ttl: 0`, which effectively means "cacheable but expires immediately." A meaningful TTL should be set.

6. **No field-level descriptions anywhere.** Schema properties have `title` but never `description`. Titles like "Id" and "Title" are just capitalized field names and add minimal value. Descriptions should explain the field's purpose, constraints, and valid values.

7. **delete_task output schema is empty.** The output schema is just `type: object` with no properties. Consumers have no idea what the response looks like.

8. **Destructive operation lacks approval gate.** `delete_task.delete` is marked `destructive: true` but `requires_approval: false`. For a permanent deletion, requiring approval is a best practice for safety.

9. **Metadata is minimal across the board.** Only `source: native` is present. Useful additions: `author`, `category`, `domain`, `sla`, `rate_limit`, `deprecated`, `since_version`.

10. **Input schema for get/delete missing titles and descriptions.** The `task_id` parameter in `get_task.get` and `delete_task.delete` lacks a `title` and `description`, unlike the same field in `update_task.put`.

---

## Recommendations

- **Immediate wins (low effort, high impact):** Fix `idempotent` flags on GET modules, add tags, remove `pagination_style` when `paginated: false`, set a real `cache_ttl` on cacheable modules, set `requires_approval: true` on delete_task.
- **Medium effort:** Write distinct, substantive `documentation` for each module. Add field-level `description` to all schema properties. Define output schema properties for delete_task.
- **Longer term:** Enrich `metadata` with author, category, and operational attributes. Add input validation constraints (minLength, minimum, enum where applicable). Consider adding error response schemas.
