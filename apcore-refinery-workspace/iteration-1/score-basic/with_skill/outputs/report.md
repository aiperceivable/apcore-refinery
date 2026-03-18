apcore-refinery score — Quality Report

Project: flask-apcore demo (flask-apcore)
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
  2. 5/5 modules have no documentation (missing or under 50 chars)
  3. 1/5 modules have default annotations (none explicitly set)
  4. 5/5 modules have no examples
  5. 5/5 modules have no tags
  6. 5/5 modules missing x-when-to-use metadata

  Dimension Breakdown:
  ─────────────────────────────────────────────────────────────────

  Description (max 25):
    All 5 modules have meaningful, non-placeholder descriptions and score
    the full 25 points. The scanner-generated descriptions are short but
    accurate one-liners (e.g., "Delete a task permanently.").

  Schema Descriptions (max 25):
    Only list_tasks.get scores 25 — it has an empty input_schema (no
    properties to describe), so it is vacuously complete. The other 4
    modules define input properties (title, description, done, task_id)
    but none include a `description` field on any property. This is the
    single highest-impact gap: 100 missing points across 4 modules.

  Documentation (max 15):
    Every module's `documentation` field is just a copy of the short
    description (15-25 chars). The threshold is > 50 chars. None qualify.
    Good documentation would explain behavior, edge cases, error codes,
    and side effects.

  Annotations (max 15):
    3 out of 5 modules have at least one non-default annotation:
      - list_tasks.get: readonly=true, cacheable=true
      - get_task.get: readonly=true, cacheable=true
      - update_task.put: idempotent=true
      - delete_task.delete: destructive=true
    Only create_task.post has all-default annotations and scores 0.
    The scanner correctly inferred several annotations from the code.

  Examples (max 10):
    No module provides any examples. Examples with title + inputs help
    LLM agents understand how to call tools and are required for A2A
    AgentSkill discovery.

  Tags (max 5):
    All modules have empty tag lists. Tags enable A2A skill filtering
    and CLI prefix-based serving (e.g., `--tags tasks`).

  Intent Metadata (max 5):
    No module has `metadata.x-when-to-use`. This field is appended to
    MCP Tool descriptions to guide LLM agents on when to select a tool.

  Recommendations:
  ─────────────────────────────────────────────────────────────────

  To raise the average score significantly, prioritize these fixes:

  1. Add `description` fields to all input_schema properties (+25 pts
     per module for create_task, get_task, update_task, delete_task).
     This alone would raise the average from 42 to 62.

  2. Write meaningful `documentation` for each module (> 50 chars) that
     explains behavior, error handling, and side effects (+15 pts each).
     This would raise the average from 62 to 77.

  3. Add at least one example per module with `title` and `inputs`
     (+10 pts each). Average would reach 87.

  4. Add tags to every module (+5 pts each). Average would reach 92.

  5. Add `metadata.x-when-to-use` to every module (+5 pts each).
     Average would reach 97.

  6. Set non-default annotations on create_task.post (+15 pts).
     Final potential average: 100.

  Tip: Run /apcore-refinery:refine to improve low-scoring modules.
