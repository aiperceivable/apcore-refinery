# Scope & Boundaries

This document defines what `apcore-refinery` **does** and **does not do**.

## What apcore-refinery Is

A **standalone, development-time CLI tool** that assesses and improves the quality of apcore module metadata. It operates on `.binding.yaml` files produced by any `apcore-toolkit` scanner and outputs enhanced versions optimized for MCP, A2A, and CLI consumption.

**Core responsibilities:**

| Responsibility | Description |
|---------------|-------------|
| **Quality scoring** | Deterministic, reproducible metadata quality assessment (0-100 per module) with per-field diagnostics |
| **Source code resolution** | Locate and read function source code from `target` strings across Python and TypeScript |
| **LLM-powered refinement** | Generate high-quality descriptions, schema annotations, behavioral flags, and protocol-specific fields using capable language models |
| **Protocol-aware generation** | Simultaneously produce metadata needed by MCP (`x-when-to-use`, `x-common-mistakes`), A2A (`examples`, `tags`), and CLI (parameter descriptions) |
| **Human review workflow** | Diff-first approach: show proposed changes before applying |
| **CI/CD integration** | Quality gate via `score --fail-under` for automated pipelines |

## What apcore-refinery Is NOT

### Not a Scanner

Scanning (discovering endpoints, extracting route metadata, parsing decorators) is `apcore-toolkit`'s responsibility. Refinery starts where scanning ends — it consumes scanner output, not source code directly.

| Dimension | apcore-toolkit scanners | apcore-refinery |
|-----------|------------------------|-----------------|
| **Input** | Framework app objects, source code | `.binding.yaml` files + source code for context |
| **Output** | `.binding.yaml` or Registry entries | Enhanced `.binding.yaml` files |
| **When** | Build time / app startup | Development time (explicit invocation) |
| **Frequency** | Every build/deploy | When code changes significantly |

### Not a Runtime Component

Refinery is invoked explicitly by developers, not embedded in application startup. Its outputs (enhanced YAML files) are committed to source control and consumed at runtime by framework adapters.

### Not a Code Modifier

Refinery **never** modifies source code. It only modifies metadata files. Specifically:

- It will not rename function parameters (even if names are poor)
- It will not add docstrings to source files
- It will not modify type annotations
- It will not generate or modify `@module` decorators

### Not a Model Provider

Refinery calls external LLM APIs but does not bundle, fine-tune, or host models. It requires the user to provide an API key for their chosen provider.

## Two Modes, One Quality Model

Both `score` and `refine` share the same understanding of "what good metadata looks like":

```
                    ┌──────────────────────────┐
                    │   QualityModel (shared)   │
                    │                          │
                    │  - Field completeness     │
                    │  - Placeholder detection  │
                    │  - Protocol requirements  │
                    │  - Schema coverage        │
                    │  - Annotation accuracy    │
                    └──────────┬───────────────┘
                               │
                    ┌──────────┼──────────┐
                    │                     │
                ┌───▼────┐          ┌─────▼───┐
                │ score  │          │ refine  │
                │        │          │         │
                │ Report │          │ Enhance │
                │ gaps   │          │ gaps    │
                └────────┘          └─────────┘
                (deterministic)     (LLM-powered)
```

`score` diagnoses problems; `refine` fixes them. If `score` says "description is a placeholder" and scores it 0/25, `refine` knows to prioritize generating a real description from source code.

## Quality Model: What "Good Metadata" Means

The scoring rubric is protocol-driven. Each field is weighted by its impact on downstream consumers:

| Field | Weight | MCP Impact | A2A Impact | CLI Impact |
|-------|--------|------------|------------|------------|
| `description` | 25 | Tool selection | Skill discovery | Command help |
| `input_schema` property descriptions | 25 | Parameter understanding | — | Flag help text |
| `documentation` | 15 | Resource context | — | — |
| `annotations` (non-default) | 15 | Safety hints | — | Approval gates |
| `examples` | 10 | — | Skill examples | — |
| `tags` | 5 | — | Skill discovery | — |
| `x-when-to-use` / `x-when-not-to-use` | 5 | Tool selection guidance | — | — |

A module scoring 100/100 has all fields populated with accurate, non-placeholder values. A module scoring 0/100 has only auto-generated placeholders and empty schemas.

## Relationship to Other Projects

```
┌─────────────────────────────────────────────────────────┐
│                    apcore (spec + SDK)                   │
│  Module protocol · Registry · Executor · Schema system  │
└──────────────────────────┬──────────────────────────────┘
                           │ consumes artifacts from
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────┴──────┐  ┌─────┴──────┐  ┌──────┴──────┐
   │ django-apcore│  │flask-apcore│  │nestjs-apcore│
   │ (adapter)   │  │ (adapter)  │  │ (adapter)   │
   └──────┬──────┘  └─────┬──────┘  └──────┬──────┘
          │                │                │
          └────────────────┼────────────────┘
                           │ all use shared logic from
                ┌──────────┴──────────┐
                │   apcore-toolkit    │
                │  scan → .binding.yaml│
                └──────────┬──────────┘
                           │ consumed by
                ┌──────────┴──────────┐
                │  apcore-refinery    │
                │  score + refine     │
                └──────────┬──────────┘
                           │ enhanced .binding.yaml consumed by
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────┴──────┐  ┌─────┴──────┐  ┌──────┴──────┐
   │  apcore-mcp │  │ apcore-a2a │  │  apcore-cli │
   │  (bridge)   │  │  (bridge)  │  │  (bridge)   │
   └─────────────┘  └────────────┘  └─────────────┘
```

## Guiding Principles

1. **Measure before fixing.** `score` works without LLM. Always know the current quality before spending tokens on refinement.
2. **Human in the loop.** Default to showing diffs. Never silently overwrite metadata.
3. **Protocol-aware, not protocol-specific.** Generate all fields in one pass. Don't make the user run separate tools for MCP vs A2A.
4. **Language-agnostic.** Consume YAML, not in-memory objects. One tool for all scanner languages.
5. **Source code is context, not input.** Read code to understand intent, but never modify it.
