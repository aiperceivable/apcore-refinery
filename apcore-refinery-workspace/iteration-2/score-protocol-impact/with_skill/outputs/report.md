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
  1. 5/5 modules have no documentation (documentation is either missing, under 50 chars, or identical to description)
  2. 4/5 modules have no parameter descriptions in input_schema (only list_tasks.get is vacuously complete with zero properties)
  3. 5/5 modules have no examples
  4. 5/5 modules have no tags
  5. 5/5 modules have no x-when-to-use intent metadata
  6. 1/5 modules (create_task.post) has all annotations at defaults despite being a POST endpoint

Scanner Degradation Warnings:
  - [description == documentation] 5/5 modules have identical description and documentation text.
    Diagnosis: The scanner did not find a distinct docstring body for any module. The documentation
    field is simply a copy of the short description, providing no additional value.
    Framework context (Flask): The route functions likely have no multi-line docstrings, or the
    scanner only extracted the first line.

  - [all annotations at defaults on POST] create_task.post has all annotations at their default
    values despite being a POST endpoint that creates data. The scanner's annotation inference
    did not differentiate this endpoint from a read-only GET.

  - [destructive without approval] delete_task.delete has destructive=true but
    requires_approval=false. This is a safety gap — a destructive operation can be executed
    without any human-in-the-loop gate.

Protocol Impact:
  MCP:  5 modules missing x-when-to-use (LLM has no guidance on when to select these tools)
        2 modules have default annotations (create_task.post — safety hints absent for MCP ToolAnnotations)
        1 module with destructive=true but requires_approval=false (delete_task.delete — safety gap)
        4 modules have no x-llm-description on input_schema properties (LLM has weaker parameter understanding)
  A2A:  5 modules have no examples (AgentSkill discovery has zero example titles)
        5 modules have no tags (skill filtering unavailable)
        1 destructive module (delete_task.delete) lacks safety signal visible to A2A
          — A2A bridge drops all annotations (known bug), so description is the ONLY safety indicator.
            Current description "Delete a task permanently." hints at permanence but does not
            explicitly state "destructive" or "irreversible."
  CLI:  4 modules have no input_schema property descriptions (--flag help text will be empty)
        — get_task.get: task_id has no description
        — create_task.post: title, description, done have no description
        — update_task.put: title, description, done, task_id have no description
        — delete_task.delete: task_id has no description

Per-Module Detail:

  ## list_tasks.get — 65/100
  Strengths:
    - Description is clear and non-placeholder (25/25)
    - No input parameters, so schema descriptions are vacuously complete (25/25)
    - Annotations correctly set readonly=true and cacheable=true (15/15)
  Weaknesses:
    - Documentation is identical to description — no additional detail (0/15)
    - No examples for A2A discovery (0/10)
    - No tags (0/5)
    - No x-when-to-use intent metadata (0/5)
  Degradation: description == documentation (scanner found no distinct docstring body)

  ## create_task.post — 25/100 (lowest)
  Strengths:
    - Description is clear and non-placeholder (25/25)
  Weaknesses:
    - 0/3 input_schema properties have descriptions (0/25)
    - Documentation is identical to description (0/15)
    - All annotations at defaults — POST endpoint not differentiated from GET (0/15)
    - No examples (0/10)
    - No tags (0/5)
    - No x-when-to-use (0/5)
  Degradation: description == documentation; all annotations at defaults on a POST endpoint

  ## get_task.get — 40/100
  Strengths:
    - Description is clear and non-placeholder (25/25)
    - Annotations correctly set readonly=true and cacheable=true (15/15)
  Weaknesses:
    - 0/1 input_schema properties (task_id) have descriptions (0/25)
    - Documentation is identical to description (0/15)
    - No examples (0/10)
    - No tags (0/5)
    - No x-when-to-use (0/5)
  Degradation: description == documentation

  ## update_task.put — 40/100
  Strengths:
    - Description is clear and non-placeholder (25/25)
    - Annotations correctly set idempotent=true for PUT semantics (15/15)
  Weaknesses:
    - 0/4 input_schema properties have descriptions (0/25)
    - Documentation is identical to description (0/15)
    - No examples (0/10)
    - No tags (0/5)
    - No x-when-to-use (0/5)
  Degradation: description == documentation

  ## delete_task.delete — 40/100
  Strengths:
    - Description is clear and non-placeholder (25/25)
    - Annotations correctly set destructive=true (15/15)
  Weaknesses:
    - 0/1 input_schema properties (task_id) have descriptions (0/25)
    - Documentation is identical to description (0/15)
    - No examples (0/10)
    - No tags (0/5)
    - No x-when-to-use (0/5)
  Degradation: description == documentation; destructive=true without requires_approval=true (safety gap)

Tip: Run /apcore-refinery:refine to improve low-scoring modules.
