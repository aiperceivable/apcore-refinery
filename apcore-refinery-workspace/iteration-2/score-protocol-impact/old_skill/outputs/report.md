apcore-refinery score — Quality Report

Project: flask-apcore demo (flask-apcore v0.3.0)
Modules: 5
Date: 2026-03-17

  Module ID                    | Total | Desc | Schema | Docs | Annot | Ex  | Tags | Intent
  ─────────────────────────────┼───────┼──────┼────────┼──────┼───────┼─────┼──────┼───────
  list_tasks.get               |   65  |  25  |   25   |   0  |  15   |   0 |   0  |   0
  create_task.post             |   25  |  25  |    0   |   0  |   0   |   0 |   0  |   0
  get_task.get                 |   40  |  25  |    0   |   0  |  15   |   0 |   0  |   0
  update_task.put              |   40  |  25  |    0   |   0  |  15   |   0 |   0  |   0
  delete_task.delete           |   40  |  25  |    0   |   0  |  15   |   0 |   0  |   0

  AVERAGE: 42/100

  Top Issues:
  1. 4/5 modules have no parameter descriptions in input_schema
  2. 5/5 modules have no documentation (documentation field is <= 50 chars)
  3. 5/5 modules have no examples
  4. 5/5 modules have no tags
  5. 5/5 modules missing x-when-to-use metadata
  6. 2/5 modules have default annotations (none explicitly set)

  Dimension Breakdown:
  - Description (max 25):   All 5 modules scored 25/25. Descriptions are present and non-placeholder.
  - Schema Descriptions (max 25):   Only list_tasks.get scored 25/25 (vacuously complete — no input properties).
                                    The other 4 modules have input properties with no `description` field (only `title` is present).
  - Documentation (max 15):   All 5 modules scored 0/15. Each has a documentation field, but it merely
                              duplicates the short description (< 50 chars). Real documentation should
                              explain behavior, edge cases, error conditions, and usage guidance.
  - Annotations (max 15):   3/5 modules scored 15/15 — list_tasks.get (readonly, cacheable),
                            get_task.get (readonly, cacheable), update_task.put (idempotent),
                            and delete_task.delete (destructive) all have at least one non-default annotation.
                            create_task.post scored 0/15 — all annotations are at their default values.
  - Examples (max 10):   0/5 modules have any examples. No module provides a title+inputs example entry.
  - Tags (max 5):   0/5 modules have tags. All have empty tag lists.
  - Intent Metadata (max 5):   0/5 modules have x-when-to-use in their metadata dict.

  Per-Module Detail:

  ### list_tasks.get (65/100)
  - Description (25/25): "List all tasks." — non-placeholder, 15 chars.
  - Schema Descriptions (25/25): No input properties (vacuously complete).
  - Documentation (0/15): "List all tasks." — only 15 chars, needs > 50.
  - Annotations (15/15): readonly=true, cacheable=true differ from defaults.
  - Examples (0/10): No examples provided.
  - Tags (0/5): Empty list.
  - Intent Metadata (0/5): No x-when-to-use key in metadata.

  ### create_task.post (25/100)
  - Description (25/25): "Create a new task." — non-placeholder, 18 chars.
  - Schema Descriptions (0/25): 3 properties (title, description, done), 0 have a `description` field. Score: 25 * 0/3 = 0.
  - Documentation (0/15): "Create a new task." — only 18 chars, needs > 50.
  - Annotations (0/15): All annotations match defaults.
  - Examples (0/10): No examples provided.
  - Tags (0/5): Empty list.
  - Intent Metadata (0/5): No x-when-to-use key in metadata.

  ### get_task.get (40/100)
  - Description (25/25): "Get a task by its ID." — non-placeholder, 21 chars.
  - Schema Descriptions (0/25): 1 property (task_id), 0 have a `description` field. Score: 25 * 0/1 = 0.
  - Documentation (0/15): "Get a task by its ID." — only 21 chars, needs > 50.
  - Annotations (15/15): readonly=true, cacheable=true differ from defaults.
  - Examples (0/10): No examples provided.
  - Tags (0/5): Empty list.
  - Intent Metadata (0/5): No x-when-to-use key in metadata.

  ### update_task.put (40/100)
  - Description (25/25): "Update an existing task." — non-placeholder, 23 chars.
  - Schema Descriptions (0/25): 4 properties (title, description, done, task_id), 0 have a `description` field. Score: 25 * 0/4 = 0.
  - Documentation (0/15): "Update an existing task." — only 23 chars, needs > 50.
  - Annotations (15/15): idempotent=true differs from default.
  - Examples (0/10): No examples provided.
  - Tags (0/5): Empty list.
  - Intent Metadata (0/5): No x-when-to-use key in metadata.

  ### delete_task.delete (40/100)
  - Description (25/25): "Delete a task permanently." — non-placeholder, 25 chars.
  - Schema Descriptions (0/25): 1 property (task_id), 0 have a `description` field. Score: 25 * 0/1 = 0.
  - Documentation (0/15): "Delete a task permanently." — only 25 chars, needs > 50.
  - Annotations (15/15): destructive=true differs from default.
  - Examples (0/10): No examples provided.
  - Tags (0/5): Empty list.
  - Intent Metadata (0/5): No x-when-to-use key in metadata.

  Tip: Run /apcore-refinery:refine to improve low-scoring modules.
