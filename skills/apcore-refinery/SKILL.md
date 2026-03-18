---
name: apcore-refinery
description: >
  Apcore refinery dashboard for *-apcore projects (flask-apcore, django-apcore,
  nestjs-apcore, tiptap-apcore). Shows project summary, module count, quality
  distribution (good/fair/poor), and available commands. Use this when the user
  wants an overview of their project's metadata health, asks "what's the status",
  "how many modules", or invokes /apcore-refinery with no subcommand. Routes to
  score/refine/inspect subcommands when specified.
---

# Apcore Refinery — Dashboard

Quick overview of a `*-apcore` project's metadata health and available refinery commands.

## When to Use

- User runs `/apcore-refinery` with no arguments
- User wants an overview before diving into score/refine/inspect
- User is new to apcore-refinery and wants to see what's available

## Command Format

```
/apcore-refinery [score|refine|inspect] [args...]
```

If a subcommand is provided, route to the corresponding skill. If no arguments, show the dashboard.

## Workflow

### Step 0: Route Subcommands

Parse `$ARGUMENTS` into `subcommand` and remaining `args`:

| Input | Action |
|-------|--------|
| `score [...]` | Invoke `apcore-refinery:score` skill with remaining args |
| `refine [...]` | Invoke `apcore-refinery:refine` skill with remaining args |
| `inspect [...]` | Invoke `apcore-refinery:inspect` skill with remaining args |
| (empty) | Continue to Step 1 (dashboard) |

### Step 1: Detect *-apcore Project

Identify the project type by checking the current working directory:

1. **Python projects**: Read `pyproject.toml` or `requirements.txt` — look for `flask-apcore`, `django-apcore`, or any `*-apcore` dependency
2. **TypeScript projects**: Read `package.json` — look for `nestjs-apcore`, `express-apcore`, or any `*-apcore` dependency
3. If no `*-apcore` dependency found: use `AskUserQuestion` — "This doesn't appear to be a *-apcore project. Which project directory should I look at?"

Store:
- `framework`: the detected framework (flask, django, nestjs, etc.)
- `integration_package`: the `*-apcore` package name
- `integration_version`: installed version

### Step 2: Find Module Definitions

Search for `.binding.yaml` files:

1. Glob `**/*.binding.yaml` in project root (max depth 3)
2. Also check common directories: `bindings/`, `modules/`, `apcore_modules/`, `output/`
3. Count total modules found

If no `.binding.yaml` files found, check if the project uses direct registry registration (no YAML output). Note this for the user.

### Step 3: Quick Quality Scan

For each `.binding.yaml` file found:
- Read it
- Check if description looks like a placeholder (starts with HTTP method + path, or "Module ")
- Check if input_schema properties have descriptions
- Compute a rough quality bucket: Good (>= 70), Fair (40-69), Poor (< 40)

### Step 4: Display Dashboard

```
apcore-refinery — Project Dashboard

Project: {project_name} ({framework}-apcore v{version})
Modules: {count} found ({yaml_count} .binding.yaml files)

Quality Overview:
  Good (>= 70):  {count}
  Fair (40-69):   {count}
  Poor (< 40):    {count}
  Average Score:  {avg}/100

Commands:

  /apcore-refinery:inspect [--fix]
      Check integration health (dependencies, config, scanner setup)

  /apcore-refinery:score [--format table|json] [--fail-under N]
      Detailed quality assessment per module (read-only)

  /apcore-refinery:refine [--apply] [--modules id1,id2] [--threshold N]
      Improve metadata using source code context

Recommended flow: inspect → score → refine
```

Use `AskUserQuestion` to ask what the user wants to do next.
