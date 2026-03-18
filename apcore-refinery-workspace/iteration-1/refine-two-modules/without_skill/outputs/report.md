# Metadata Refinement Report — flask-apcore demo

**Date:** 2026-03-17
**Method:** Manual refinement (without apcore-refinery skill)
**Source:** `/Users/tercelyi/Workspace/aipartnerup/flask-apcore/examples/demo/app.py`
**Modules refined:** 2

---

## Module 1: `create_task.post`

### Score Estimate (Before / After)

| Dimension | Max | Before | After | Notes |
|-----------|-----|--------|-------|-------|
| Description | 25 | 25 | 25 | Was "Create a new task." (short but valid); now more specific |
| Schema Descriptions | 25 | 0 | 25 | No property descriptions before; all 3 input + 4 output properties now described |
| Documentation | 15 | 0 | 15 | Was identical to description; now a multi-sentence explanation |
| Annotations | 15 | 0 | 15 | `open_world` changed from `true` to `false` (no external calls — in-memory only) |
| Examples | 10 | 0 | 10 | Added 2 realistic examples with inputs and outputs |
| Tags | 5 | 0 | 5 | Added `[tasks, create, crud]` |
| Intent Metadata | 5 | 0 | 5 | Added `x-when-to-use`, `x-when-not-to-use`, `x-common-mistakes` |
| **Total** | **100** | **25** | **100** | |

### Changes

```diff
  description:
-   Create a new task.
+   Create a new task with a title, optional description, and completion status.

  documentation:
-   Create a new task.
+   Creates a new task in the in-memory task store and returns the created task
+   with its auto-assigned ID. The title is required; description defaults to an
+   empty string and done defaults to false. Each new task receives a
+   monotonically increasing integer ID. This is a write operation with no
+   external side effects beyond mutating the in-memory store. Returns the full
+   Task object including the generated ID.

  tags:
-   []
+   [tasks, create, crud]

  annotations:
-   open_world: true
+   open_world: false

  metadata:
+   x-when-to-use: Use this when the user wants to add a new task to the task list.
+   x-when-not-to-use: Do not use this to update an existing task — use update_task.put instead.
+   x-common-mistakes: Forgetting that title is required; passing an integer ID (IDs are auto-assigned and must not be provided as input).

  examples:
+   - title: Create a simple task
+     inputs: { title: "Buy groceries" }
+     output: { id: 3, title: "Buy groceries", description: "", done: false }
+   - title: Create a task with description and done status
+     inputs: { title: "Write quarterly report", description: "Cover Q1 sales figures and projections", done: false }
+     output: { id: 4, title: "Write quarterly report", description: "Cover Q1 sales figures and projections", done: false }

  input_schema.properties.title:
+   description: The name of the task to create. This is the primary identifier shown in task listings.

  input_schema.properties.description:
+   description: Optional free-text details about the task. Defaults to an empty string if not provided.

  input_schema.properties.done:
+   description: Whether the task should be marked as completed upon creation. Defaults to false.

  output_schema.properties.id:
+   description: The auto-assigned unique identifier for the created task.

  output_schema.properties.title:
+   description: The title of the created task.

  output_schema.properties.description:
+   description: The description of the created task.

  output_schema.properties.done:
+   description: Whether the created task is marked as completed.
```

---

## Module 2: `delete_task.delete`

### Score Estimate (Before / After)

| Dimension | Max | Before | After | Notes |
|-----------|-----|--------|-------|-------|
| Description | 25 | 25 | 25 | Was "Delete a task permanently." (valid); now more descriptive |
| Schema Descriptions | 25 | 0 | 25 | `task_id` now has a description; output schema property described |
| Documentation | 15 | 0 | 15 | Was identical to description; now covers behavior, errors, and irreversibility |
| Annotations | 15 | 15 | 15 | `destructive: true` already set (non-default); additionally set `requires_approval: true` and `open_world: false` |
| Examples | 10 | 0 | 10 | Added 1 example with input and output |
| Tags | 5 | 0 | 5 | Added `[tasks, delete, crud]` |
| Intent Metadata | 5 | 0 | 5 | Added `x-when-to-use`, `x-when-not-to-use`, `x-common-mistakes` |
| **Total** | **100** | **40** | **100** | |

### Changes

```diff
  description:
-   Delete a task permanently.
+   Permanently delete a task by its ID, removing it from the task list.

  documentation:
-   Delete a task permanently.
+   Permanently removes a task from the in-memory task store by its integer ID.
+   Returns {"deleted": true} on success. Returns a 404 error with
+   {"error": "not found"} if no task with the given ID exists. This operation
+   is irreversible — the deleted task cannot be recovered. The ID is not reused
+   for future tasks.

  tags:
-   []
+   [tasks, delete, crud]

  annotations:
-   open_world: true
+   open_world: false
-   requires_approval: false
+   requires_approval: true

  metadata:
+   x-when-to-use: Use this when the user explicitly asks to remove or delete a specific task.
+   x-when-not-to-use: Do not use this to mark a task as completed — use update_task.put with done=true instead. Do not use without confirming the task ID exists first via get_task.get or list_tasks.get.
+   x-common-mistakes: Deleting a task without first confirming with the user, since this action is permanent and cannot be undone.

  examples:
+   - title: Delete a task by ID
+     inputs: { task_id: 1 }
+     output: { deleted: true }

  input_schema.properties.task_id:
+   description: The unique integer ID of the task to delete. Must correspond to an existing task; a 404 error is returned if the task does not exist.

  output_schema:
-   type: object
+   type: object
+   properties:
+     deleted:
+       type: boolean
+       description: True if the task was successfully deleted.
```

---

## Summary

| Module | Before | After | Delta |
|--------|--------|-------|-------|
| `create_task.post` | 25 | 100 | +75 |
| `delete_task.delete` | 40 | 100 | +60 |
| **Average** | **32** | **100** | **+68** |

### Key Decisions

1. **`open_world` set to `false` on both modules** — the source code operates entirely on an in-memory dict; no external HTTP calls, database queries, file I/O, or subprocesses.

2. **`requires_approval` set to `true` on `delete_task.delete`** — deletion is permanent and irreversible. When the approval system is enabled (`APCORE_SERVE_APPROVAL="elicit"`), this annotation triggers human confirmation before execution.

3. **`idempotent` left `false` on both** — `create_task.post` generates a new ID each call (not idempotent). `delete_task.delete` returns 404 on the second call (not idempotent).

4. **Output schema enriched for `delete_task.delete`** — the original had a bare `type: object` with no properties. Added the `deleted` boolean property to match the actual `{"deleted": True}` return value.
