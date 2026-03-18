# Apcore Refinery -- Refine Report

**Project:** flask-apcore demo (Task Manager API)
**Source:** `/Users/tercelyi/Workspace/aipartnerup/flask-apcore/examples/demo/app.py`
**Modules refined:** 2

---

## Modules to refine (2 below threshold 70):

| Module ID              | Current Score |
|------------------------|---------------|
| create_task.post       | 25            |
| delete_task.delete     | 40            |

Modules skipped (3 at or above threshold... actually all below, but not in scope):
  - list_tasks.get, get_task.get, update_task.put (not requested)

---

## Diff: create_task.post (25 -> 100)

```
  description:
  - "Create a new task."
  + "Create a new task with a title, optional description, and completion status"

  input_schema.properties.title:
  + description: "The name or summary of the task to create (required)."

  input_schema.properties.description:
  + description: "An optional longer explanation or details about the task."

  input_schema.properties.done:
  + description: "Whether the task is already completed at creation time. Defaults to false."

  annotations:
  ~ open_world: true -> false

  documentation:
  - "Create a new task."
  + |
  +   Creates a new task in the in-memory task store and returns the complete task
  +   object including its auto-assigned ID. The title field is required; description
  +   defaults to an empty string and done defaults to false. Each task receives a
  +   unique sequential integer ID. Returns the created task as a Task object with
  +   all fields populated. No authentication is required.

  + examples:
  +   - title: "Create a simple task"
  +     inputs: {title: "Buy groceries"}
  +     output: {id: 3, title: "Buy groceries", description: "", done: false}
  +   - title: "Create a task with description and pre-marked done"
  +     inputs: {title: "Write project proposal", description: "Draft the Q3 project proposal for the engineering team", done: true}
  +     output: {id: 4, title: "Write project proposal", description: "Draft the Q3 project proposal for the engineering team", done: true}

  + tags: [tasks, create]

  + metadata:
  +   x-when-to-use: "Use this when the user wants to add a new task to the task list."
  +   x-when-not-to-use: "Do not use this to modify an existing task; use update_task.put instead."
  +   x-common-mistakes: "Omitting the required title field or passing a task_id (IDs are auto-assigned)."
```

### Annotation Rationale (create_task.post)

- **open_world: false** -- The function only writes to an in-memory dict. It makes no HTTP calls, reads no files, queries no databases, and runs no subprocesses. Changed from the scanner default of `true`.
- **destructive: false** -- Creating a task does not delete or remove anything.
- **idempotent: false** -- Each call creates a new task with a new ID, even with identical inputs.
- **requires_approval: false** -- Creating a task is a low-risk operation.

---

## Diff: delete_task.delete (40 -> 100)

```
  description:
  - "Delete a task permanently."
  + "Permanently delete a task by its ID, removing it from the task list"

  input_schema.properties.task_id:
  + description: "The unique integer ID of the task to delete. Use list_tasks.get or get_task.get to find valid IDs."

  annotations:
  + requires_approval: true
  ~ open_world: true -> false

  documentation:
  - "Delete a task permanently."
  + |
  +   Permanently removes a task from the in-memory task store by its integer ID.
  +   This operation is destructive and cannot be undone. Returns {"deleted": true}
  +   on success. Returns a 404 error with {"error": "not found"} if no task with
  +   the given ID exists. No authentication is required.

  + examples:
  +   - title: "Delete an existing task"
  +     inputs: {task_id: 1}
  +     output: {deleted: true}

  + tags: [tasks, delete]

  + metadata:
  +   x-when-to-use: "Use this when the user explicitly asks to remove or delete a specific task."
  +   x-when-not-to-use: "Do not use this to mark a task as completed; use update_task.put to set done to true instead."
  +   x-common-mistakes: "Passing a task title instead of the integer task_id, or assuming deletion can be undone."
```

### Annotation Rationale (delete_task.delete)

- **destructive: true** -- Already correctly set by the scanner. The function permanently removes a task via `del _tasks[task_id]`.
- **requires_approval: true** -- Deletion is irreversible. An LLM agent should confirm with the user before proceeding. This triggers MCP's `_meta.requiresApproval` and CLI's interactive confirmation prompt.
- **open_world: false** -- The function only operates on the in-memory dict. No external calls.
- **idempotent: false** -- Calling delete on an already-deleted task returns a 404, so the behavior differs on repeat calls.

---

## Summary

| Module ID              | Before | After | Delta |
|------------------------|--------|-------|-------|
| create_task.post       | 25     | 100   | +75   |
| delete_task.delete     | 40     | 100   | +60   |

**Average: 32 -> 100 (+68)**

---

## Output Files

- `create_task.post.binding.yaml` -- Proposed refined YAML for the create_task module
- `delete_task.delete.binding.yaml` -- Proposed refined YAML for the delete_task module

These are proposals only. No files in the demo project were modified.
