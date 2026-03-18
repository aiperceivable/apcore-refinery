# Idea Draft: apcore-refinery

**Status**: VALIDATED (Critical gap in metadata quality for MCP/A2A/CLI consumption)
**Owner**: Spec Forge
**Date**: 2026-03-16

---

## 1. The Core Idea

`apcore-refinery` is a standalone, language-agnostic CLI tool that transforms low-quality module metadata (produced by `apcore-toolkit` scanners) into high-quality, protocol-aware metadata optimized for consumption by LLM agents (MCP), AI agents (A2A), and developers (CLI). It combines deterministic quality scoring with LLM-powered semantic refinement.

## 2. Problem & Validation

### The "Why now?"

- **Scanner output quality is insufficient**: Static scanning (AST, regex, type hints) captures structure but not semantics. Functions with poor docstrings, generic parameter names (`data`, `p1`, `args`), or missing type hints produce metadata that LLMs cannot reliably reason about.
- **Different protocols need different metadata**: MCP relies on `description` + `input_schema` + `x-when-to-use`; A2A relies on `tags` + `examples`; CLI relies on parameter descriptions. No single scanner can optimize for all three simultaneously.
- **The existing AIEnhancer in apcore-toolkit is ineffective**: It uses a 0.6B parameter local model with no source code context, only fills empty fields (not inaccurate ones), and silently overwrites metadata with no human review.
- **The problem scales with adoption**: As more legacy projects are onboarded to apcore, the percentage of poorly-documented modules grows. Manual metadata authoring does not scale.
- **Powerful LLMs now exist**: Claude Opus 4.6 and similar models are capable of reading source code and generating accurate, structured metadata — but this capability needs to be properly integrated into the workflow.

### Validation Evidence

- Analysis of flask-apcore scanning pipeline shows routes without docstrings fall back to `"GET /api/users"` as description — useless for LLM tool selection.
- Analysis of nestjs-apcore shows schema extraction failures silently produce empty schemas with no warnings.
- MCP bridge loses `examples` and `tags`; A2A bridge loses `annotations`, `documentation`, `input_schema` details, and `x-*` metadata. Each protocol needs different fields optimized.
- The existing AIEnhancer in apcore-toolkit has been identified as architecturally misplaced: LLM calls are non-deterministic, expensive, and should not be part of a deterministic scan pipeline.

### Competitive / Alternative Analysis

| Alternative | Why insufficient |
|------------|-----------------|
| Manual metadata authoring | Does not scale; developers skip it |
| apcore-toolkit AIEnhancer | No source code context, tiny model, fill-gaps-only, no review workflow |
| Generic code documentation tools (e.g., Copilot docstrings) | Generate docstrings but not structured JSON Schema, annotations, or protocol-specific fields |
| OpenAPI spec generation | Only covers HTTP endpoints; misses annotations, examples, x-* intent metadata |

## 3. Scope & Constraints

### In-Scope

- **FR-001**: Deterministic metadata quality scoring (0-100) per module, with per-field breakdown.
- **FR-002**: LLM-powered metadata refinement using source code context (via `target` resolution + `inspect.getsource` or file read).
- **FR-003**: Protocol-aware generation — simultaneously produce fields needed by MCP (`x-when-to-use`, `x-when-not-to-use`, `x-common-mistakes`), A2A (`examples`, `tags`), and CLI (parameter `description`).
- **FR-004**: Human review workflow — `--diff` mode shows proposed changes; `--apply` commits them.
- **FR-005**: Language-agnostic input/output — consumes `.binding.yaml` files produced by any language's scanner.
- **FR-006**: Source code resolution — locate and read function source from `target` field (Python `module:qualname`, TypeScript `file:export`).
- **FR-007**: Caching — avoid redundant LLM calls for unchanged modules (hash-based).
- **FR-008**: Configurable LLM provider — default to Claude API, support OpenAI-compatible endpoints.
- **FR-009**: Batch processing — efficiently process large module sets with progress reporting.
- **FR-010**: CI/CD integration — `score` command with `--fail-under` threshold for quality gates.

### Out-of-Scope (for MVP)

- Parameter renaming or function signature modification (Phase 2 — requires runtime mapping layer).
- Web UI for review workflow (Phase 2).
- Automatic error code inference from exception handling (low ROI).
- Direct integration into scanner pipelines — refinery is always a separate, explicit step.
- Model fine-tuning or custom model training.

## 4. Requirement IDs (Initial)

### Functional Requirements

- **FR-SCORE-001**: Score module metadata quality on a 0-100 scale with deterministic, reproducible results.
- **FR-SCORE-002**: Detect placeholder descriptions (auto-generated from HTTP method + URL, function name, etc.).
- **FR-SCORE-003**: Measure input_schema property description coverage (% of properties with descriptions).
- **FR-SCORE-004**: Evaluate annotation completeness (are readonly/destructive/idempotent explicitly set vs default).
- **FR-SCORE-005**: Report scoring results in JSON, table, or Markdown format.
- **FR-REFINE-001**: Resolve `target` string to source code for Python (`module:qualname`) and TypeScript (`file:export`) modules.
- **FR-REFINE-002**: Build protocol-aware LLM prompts that simultaneously request MCP, A2A, and CLI metadata.
- **FR-REFINE-003**: Parse LLM responses and merge enhancements into existing `.binding.yaml` files.
- **FR-REFINE-004**: Generate unified diff output for human review before applying changes.
- **FR-REFINE-005**: Cache LLM responses keyed by source code hash + YAML hash to avoid redundant calls.
- **FR-REFINE-006**: Support `--apply` mode to write enhanced YAML back to disk.
- **FR-REFINE-007**: Tag all LLM-generated fields with `x-refined-by: apcore-refinery` and timestamp.

### Non-Functional Requirements

- **NFR-001**: Scoring must complete in < 1 second per module (no external calls).
- **NFR-002**: Refinery must support processing 100+ modules in a single run with progress reporting.
- **NFR-003**: LLM API key must never be logged or included in output artifacts.
- **NFR-004**: Must work with `.binding.yaml` files from any scanner (flask-apcore, django-apcore, nestjs-apcore, etc.) without knowing the source framework.

---

## 5. "What if we don't build this?"

Module metadata quality remains the weakest link in the apcore ecosystem. LLM agents will receive tool definitions with placeholder descriptions (`"POST /api/users"`), empty schemas, and missing behavioral annotations — leading to incorrect tool selection, hallucinated parameters, and potentially dangerous operations executed without approval gates. The gap between "code exists" and "code is AI-Perceivable" will widen as more legacy projects are onboarded, and developers will blame apcore rather than their own documentation practices.
