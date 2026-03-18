# apcore-refinery

> Part of the [apcore](https://github.com/aipartnerup/apcore) ecosystem

Metadata quality assessment and refinement skill for Claude Code. Helps `*-apcore` project developers (flask-apcore, django-apcore, nestjs-apcore) improve their module metadata for MCP, A2A, and CLI consumption.

## Commands

| Command | Usage | Description |
|---------|-------|-------------|
| `/apcore-refinery` | | Project dashboard — module count, quality summary, available commands |
| `/apcore-refinery:score` | `[path] [--format table\|json] [--fail-under N]` | Assess module metadata quality (read-only, deterministic) |
| `/apcore-refinery:refine` | `[path] [--apply] [--modules id1,id2] [--threshold N]` | Refine metadata using source code context (diff-first, human-in-the-loop) |
| `/apcore-refinery:inspect` | `[path] [--fix]` | Check *-apcore integration health (dependencies, config, scanner setup) |

## Target Users

Developers using framework integrations:
- `flask-apcore` — Flask integration
- `django-apcore` — Django integration
- `nestjs-apcore` — NestJS integration
- Any future `{framework}-apcore` integration

## The Problem

Static scanners extract metadata from code, but quality depends on how well the code is documented. In practice:

- Legacy functions produce placeholder descriptions like `"POST /api/users"`
- Parameters named `data`, `p1`, `args` carry no semantic meaning
- `input_schema` properties have types but no descriptions
- Behavioral annotations (`destructive`, `readonly`) default to `false` regardless of actual behavior
- Protocol-specific fields (`x-when-to-use`, `examples`, `tags`) are never generated

An LLM agent receiving these tool definitions cannot reliably decide **when** to call a tool, **what** to pass, or **whether** the operation is safe.

## How It Works

The recommended flow is: **inspect → score → refine**

1. **inspect** — Is your `*-apcore` integration set up correctly? Checks dependencies, scanner config, output setup, and environment settings.

2. **score** — How good is your module metadata? Applies a deterministic 0-100 scoring rubric covering 7 dimensions: description quality, schema descriptions, documentation, annotations, examples, tags, and intent metadata.

3. **refine** — Let Claude read your source code and generate high-quality metadata optimized for MCP, A2A, and CLI consumption simultaneously. Shows a diff for human review before writing changes.

No external LLM API calls needed — Claude Code itself performs the refinement.

## Architecture

```
Source Code ──────────────────────┐
                                  │
Scanner (apcore-toolkit)          │
    │                             │
    ▼                             ▼
.binding.yaml ──────────► apcore-refinery
(raw, may be low quality)    │    │    │
                             │    │    │
                     inspect │  score  │ refine
                             │    │    │
                             ▼    ▼    ▼
                     health  quality  .binding.yaml
                     report  report   (enhanced)
                                          │
                               ┌──────────┼──────────┐
                               │          │          │
                           apcore-mcp apcore-a2a apcore-cli
```

## Prerequisites

1. A `*-apcore` project with scanned modules (`.binding.yaml` files)
2. Source code accessible for `refine` (needed to read function implementations)

## Integration with Other Skills

- **apcore-skills** — For apcore ecosystem maintainers (core SDKs, bridges). apcore-refinery is for `*-apcore` end users.
- **code-forge:tdd** — After refine identifies code quality issues, use TDD to improve source code
- **spec-forge:audit** — For auditing documentation quality beyond module metadata

## Documentation

- **[Idea Draft](ideas/draft.md)** — Problem validation and initial requirements
- **[Scope & Boundaries](docs/scope.md)** — What refinery does and does not do

## License

Apache-2.0
