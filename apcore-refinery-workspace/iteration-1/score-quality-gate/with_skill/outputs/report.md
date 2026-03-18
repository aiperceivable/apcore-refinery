# apcore-refinery score -- Quality Report

Project: flask-apcore demo (flask-apcore v0.3.0)
Modules: 5
Date: 2026-03-17

```
  Module ID                    | Total | Desc | Schema | Docs | Annot | Ex  | Tags | Intent
  -----------------------------+-------+------+--------+------+-------+-----+------+-------
  list_tasks.get               |   65  |  25  |   25   |   0  |  15   |   0 |   0  |   0
  create_task.post             |   25  |  25  |    0   |   0  |   0   |   0 |   0  |   0
  get_task.get                 |   40  |  25  |    0   |   0  |  15   |   0 |   0  |   0
  update_task.put              |   40  |  25  |    0   |   0  |  15   |   0 |   0  |   0
  delete_task.delete           |   40  |  25  |    0   |   0  |  15   |   0 |   0  |   0
```

  AVERAGE: 42/100

  Top Issues:
  1. 4/5 modules have no parameter descriptions in input_schema
  2. 5/5 modules have no documentation (or documentation under 50 characters)
  3. 5/5 modules have no examples
  4. 5/5 modules have no tags
  5. 5/5 modules missing x-when-to-use metadata
  6. 1/5 modules have default annotations (none explicitly set)

---

## Quality Gate Result

```
Quality gate FAILED: 42/100 < 60

5 modules below threshold. Run /apcore-refinery:refine to improve.
```

## Dimension Breakdown

### list_tasks.get (65/100)
- **Description (25/25):** "List all tasks." -- non-placeholder, 16 chars
- **Schema Descriptions (25/25):** No input properties (vacuously complete)
- **Documentation (0/15):** "List all tasks." -- only 15 chars (needs > 50)
- **Annotations (15/15):** readonly=true, cacheable=true differ from defaults
- **Examples (0/10):** No examples provided
- **Tags (0/5):** Empty list
- **Intent Metadata (0/5):** No x-when-to-use in metadata

### create_task.post (25/100)
- **Description (25/25):** "Create a new task." -- non-placeholder, 18 chars
- **Schema Descriptions (0/25):** 0/3 properties have descriptions (title, description, done)
- **Documentation (0/15):** "Create a new task." -- only 18 chars (needs > 50)
- **Annotations (0/15):** All annotations match defaults
- **Examples (0/10):** No examples provided
- **Tags (0/5):** Empty list
- **Intent Metadata (0/5):** No x-when-to-use in metadata

### get_task.get (40/100)
- **Description (25/25):** "Get a task by its ID." -- non-placeholder, 21 chars
- **Schema Descriptions (0/25):** 0/1 properties have descriptions (task_id)
- **Documentation (0/15):** "Get a task by its ID." -- only 21 chars (needs > 50)
- **Annotations (15/15):** readonly=true, cacheable=true differ from defaults
- **Examples (0/10):** No examples provided
- **Tags (0/5):** Empty list
- **Intent Metadata (0/5):** No x-when-to-use in metadata

### update_task.put (40/100)
- **Description (25/25):** "Update an existing task." -- non-placeholder, 24 chars
- **Schema Descriptions (0/25):** 0/4 properties have descriptions (title, description, done, task_id)
- **Documentation (0/15):** "Update an existing task." -- only 24 chars (needs > 50)
- **Annotations (15/15):** idempotent=true differs from defaults
- **Examples (0/10):** No examples provided
- **Tags (0/5):** Empty list
- **Intent Metadata (0/5):** No x-when-to-use in metadata

### delete_task.delete (40/100)
- **Description (25/25):** "Delete a task permanently." -- non-placeholder, 26 chars
- **Schema Descriptions (0/25):** 0/1 properties have descriptions (task_id)
- **Documentation (0/15):** "Delete a task permanently." -- only 26 chars (needs > 50)
- **Annotations (15/15):** destructive=true differs from defaults
- **Examples (0/10):** No examples provided
- **Tags (0/5):** Empty list
- **Intent Metadata (0/5):** No x-when-to-use in metadata

---

Tip: Run /apcore-refinery:refine to improve low-scoring modules.
