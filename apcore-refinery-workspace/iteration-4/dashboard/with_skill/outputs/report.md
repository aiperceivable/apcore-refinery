apcore-refinery — Project Dashboard

Project: flask-apcore demo (flask-apcore v0.3.0)
Modules: 6 found (5 .binding.yaml files, 1 @module decorator)

Quality Overview:
  Good (>= 70):  0
  Fair (40-69):   5
  Poor (< 40):    0
  Average Score:  48/100

Module Breakdown:

  | Module                | Score | Bucket | Notes                                          |
  |-----------------------|-------|--------|-------------------------------------------------|
  | list_tasks.get        | 55    | Fair   | No tags; documentation duplicates description   |
  | create_task.post      | 50    | Fair   | Input properties lack descriptions; no tags     |
  | get_task.get          | 45    | Fair   | task_id input lacks title/description; no tags  |
  | update_task.put       | 50    | Fair   | Input properties lack descriptions; no tags     |
  | delete_task.delete    | 40    | Fair   | Bare output schema; task_id lacks docs; no tags |

  Note: task_stats.v1 is registered via @module decorator (no .binding.yaml).
  It was not scored — run `inspect` to check its registration health.

Commands:

  /apcore-refinery:inspect [--fix]
      Check integration health (dependencies, config, scanner setup)

  /apcore-refinery:score [--format table|json] [--fail-under N]
      Detailed quality assessment per module (read-only)

  /apcore-refinery:refine [--apply] [--modules id1,id2] [--threshold N]
      Improve metadata using source code context

Recommended flow: inspect → score → refine
