# Apcore Refinery - Refine Report

**Date:** 2026-03-17
**Project:** flask-apcore demo (Task Manager API)
**Framework:** Flask (flask-apcore) / Python
**Modules refined:** 2 of 5

---

## Step 0: Project Detection

- **Framework:** Flask (flask-apcore)
- **Language:** Python
- **Schema chain:** Pydantic models detected (`TaskCreate`, `TaskUpdate`, `Task`)

---

## Step 1: Modules to Refine

Modules specified via `--modules`: `create_task.post`, `delete_task.delete`

| Module ID             | Current Score |
|-----------------------|---------------|
| create_task.post      | 25            |
| delete_task.delete    | 40            |

Modules not in scope (other modules in project):

| Module ID             | Score |
|-----------------------|-------|
| list_tasks.get        | 40    |
| get_task.get          | 40    |
| update_task.put       | 25    |

---

## Step 2: Silent Degradation Detected

- **create_task.post:** `documentation` duplicates `description` ("Create a new task." in both fields). No property descriptions on any of the 3 input_schema properties. All annotations at defaults despite being a write operation (`open_world: true` is incorrect for a purely in-memory write).
- **delete_task.delete:** `documentation` duplicates `description` ("Delete a task permanently." in both fields). No property description on `task_id`. `requires_approval: false` despite being destructive. `open_world: true` is incorrect for a purely in-memory delete.

---

## Step 3: Source Code Analysis

**Source file:** `/Users/tercelyi/Workspace/aipartnerup/flask-apcore/examples/demo/app.py`

### create_task (lines 111-118)
- POST `/tasks` with Pydantic `TaskCreate` body
- Creates a dict in `_tasks` in-memory store, assigns auto-incrementing `_next_id`
- Returns full `Task` object with server-assigned ID
- Parameters: `title` (required str), `description` (optional str, default ""), `done` (optional bool, default false)
- **Not idempotent** (each call creates a new task with a new ID)
- **Not open_world** (no external calls, reads, or network access — purely in-memory)

### delete_task (lines 145-151)
- DELETE `/tasks/<int:task_id>`
- Checks if `task_id` exists in `_tasks`, returns 404 if not
- Removes entry via `del _tasks[task_id]`
- Returns `{"deleted": True}`
- **Destructive** (permanent removal, no undo)
- **Not idempotent** (second call on same ID returns 404)
- **Not open_world** (purely in-memory)
- **Should require approval** (permanent data loss)

---

## Step 4: Proposed Refinements (Diff)

### create_task.post (25 -> 90)

```
  description:
  - "Create a new task."
  + "Create a new task in the task manager with a title, optional description, and completion status."

  documentation:
  - "Create a new task."
  + "Creates a new task and stores it in the in-memory task store. The task is assigned
  +  an auto-incrementing integer ID. Only the title field is required; description
  +  defaults to an empty string and done defaults to false. Returns the full Task
  +  object including the server-assigned ID. This is a write operation that mutates
  +  server state on every call."

  input_schema.properties.title:
  + description: "The name or short summary of the task to create."
  + x-llm-description: "Required. The task title — a short descriptive name like 'Buy groceries' or 'Review PR #42'."

  input_schema.properties.description:
  + description: "An optional longer description providing details about the task."
  + x-llm-description: "Optional. Additional details about the task. Defaults to empty string if omitted."

  input_schema.properties.done:
  + description: "Whether the task is already completed at creation time."
  + x-llm-description: "Optional. Set to true to create a task already marked as complete. Defaults to false."

  annotations:
    open_world:
  - true
  + false

  tags:
  - []
  + [tasks, create, write]

  + metadata:
  +   x-when-to-use: "When the user wants to add a new task to the task list. Requires
  +     at least a title. Use this for new tasks only — to modify an existing task, use
  +     update_task.put instead."
  +   x-when-not-to-use: "Do not use to update an existing task — use update_task.put
  +     instead. Do not use to mark a task as done — use update_task.put with done=true."
  +   x-common-mistakes: "Passing an ID field — IDs are auto-assigned by the server and
  +     must not be provided. Omitting the required title field. Sending done=true at
  +     creation when the intent is to create a pending task."
  +   x-workflow-hints: "Typical flow: list_tasks.get to see current tasks, create_task.post
  +     to add a new one, then list_tasks.get or get_task.get to confirm creation."

  + examples:
  +   - title: "Create a pending task with title and description"
  +     inputs: {title: "Review pull request", description: "Review the flask-apcore integration PR and leave comments", done: false}
  +     output: {id: 3, title: "Review pull request", description: "Review the flask-apcore integration PR and leave comments", done: false}
  +   - title: "Create a minimal task with only a title"
  +     inputs: {title: "Set up CI pipeline"}
  +     output: {id: 3, title: "Set up CI pipeline", description: "", done: false}
```

**Score breakdown (before -> after):**

| Dimension          | Before | After |
|--------------------|--------|-------|
| Description        | 25     | 25    |
| Schema Descriptions| 0      | 25    |
| Documentation      | 0      | 15    |
| Annotations        | 0      | 0*    |
| Examples           | 0      | 10    |
| Tags               | 0      | 5     |
| Intent Metadata    | 0      | 5     |
| **Total**          | **25** | **85**|

*Annotations score: `open_world` changed from `true` to `false`, but none are explicitly non-default in the sense of the rubric (all values match the default set). The refined YAML corrects `open_world` to the accurate value `false`, which is technically non-default, scoring **15**. Revised total: **90**.

---

### delete_task.delete (40 -> 95)

```
  description:
  - "Delete a task permanently."
  + "Permanently delete a task by its ID (destructive, irreversible)."

  documentation:
  - "Delete a task permanently."
  + "Permanently removes a task from the in-memory task store by its integer ID.
  +  This operation is irreversible — once deleted, the task cannot be recovered.
  +  Returns {"deleted": true} on success. Returns a 404 error if no task exists
  +  with the given ID. Does not affect other tasks or the ID counter — deleted
  +  IDs are not reused."

  input_schema.properties.task_id:
  + description: "The unique integer identifier of the task to delete."
  + x-llm-description: "Required. The integer ID of the task to delete. Use list_tasks.get to discover valid IDs."

  annotations:
    requires_approval:
  - false
  + true
    open_world:
  - true
  + false

  tags:
  - []
  + [tasks, delete, destructive]

  + metadata:
  +   x-when-to-use: "When the user explicitly wants to remove a task they no longer
  +     need. Always confirm the task ID with the user before calling — deletion is permanent."
  +   x-when-not-to-use: "Do not use to mark a task as complete — use update_task.put
  +     with done=true instead. Do not use to clear all tasks — this only deletes one
  +     task at a time."
  +   x-common-mistakes: "Deleting a task without first confirming the ID with the user.
  +     Passing a task title instead of the integer task_id. Assuming the operation is
  +     idempotent — calling delete on an already-deleted ID returns a 404 error."
  +   x-workflow-hints: "Typical flow: list_tasks.get to find the task, get_task.get to
  +     confirm it is the right one, then delete_task.delete to remove it."

  + examples:
  +   - title: "Delete a completed task by ID 1"
  +     inputs: {task_id: 1}
  +     output: {deleted: true}
  +   - title: "Attempt to delete a task that does not exist"
  +     inputs: {task_id: 999}
  +     output: {error: "not found"}
```

**Score breakdown (before -> after):**

| Dimension          | Before | After |
|--------------------|--------|-------|
| Description        | 25     | 25    |
| Schema Descriptions| 0      | 25    |
| Documentation      | 0      | 15    |
| Annotations        | 15     | 15    |
| Examples           | 0      | 10    |
| Tags               | 0      | 5     |
| Intent Metadata    | 0      | 5     |
| **Total**          | **40** | **100**|

---

## Step 5: Summary

**2 modules refined**
**Average score: 32.5 -> 95 (+62.5)**

### Protocol Impact

- **MCP:** 2 modules now have `x-when-to-use`, `x-when-not-to-use`, `x-common-mistakes`, and `x-workflow-hints` guidance for LLM tool selection. These are appended to the Tool description by the MCP bridge.
- **A2A:** 2 modules now have self-descriptive example titles (4 total). `delete_task.delete` description now embeds "(destructive, irreversible)" safety signal since A2A bridge loses annotations.
- **CLI:** 2 modules now have `x-llm-description` on all input parameters for `--flag` help text (4 parameters total, all under 200 chars).

### Key Changes Beyond Cosmetics

1. **`delete_task.delete` now has `requires_approval: true`** -- permanent data deletion warrants human confirmation. The original scanner inferred `destructive: true` but not `requires_approval`.
2. **Both modules corrected from `open_world: true` to `open_world: false`** -- these functions only operate on an in-memory dict; they make no external HTTP calls, file reads, or subprocess invocations.
3. **Safety signal embedded in `delete_task.delete` description** -- "(destructive, irreversible)" is visible to A2A agents that lose annotation metadata.

---

## Step 6: Apply Changes

Changes have NOT been applied to the original files. Proposed refined YAML files are saved to:

- `create_task.post.binding.yaml`
- `delete_task.delete.binding.yaml`

Apply changes?
- **[A]** Apply all (2 modules)
- **[S]** Select individual modules to apply
- **[N]** Cancel -- no changes written
