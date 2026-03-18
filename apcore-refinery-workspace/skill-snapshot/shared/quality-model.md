## Quality Model

This shared resource defines how to assess apcore module metadata quality. Both `score` and `refine` use these same rules — score diagnoses, refine fixes.

### Scoring Dimensions (100 points total)

| Dimension | Max | How to Score |
|-----------|-----|-------------|
| **Description** | 25 | 25 if non-placeholder and >= 10 chars; 0 if placeholder or missing |
| **Schema Descriptions** | 25 | `25 * (properties_with_description / total_properties)`. If no properties: 25 (vacuously complete). |
| **Documentation** | 15 | 15 if non-null and > 50 chars; 0 otherwise |
| **Annotations** | 15 | 15 if at least one annotation is explicitly non-default; 0 if all are default (`readonly=false, destructive=false, idempotent=false, requires_approval=false, open_world=true, streaming=false, cacheable=false, paginated=false`) |
| **Examples** | 10 | 10 if at least 1 example with title + inputs; 0 otherwise |
| **Tags** | 5 | 5 if non-empty list; 0 otherwise |
| **Intent Metadata** | 5 | 5 if `metadata` contains `x-when-to-use`; 0 otherwise |

### Placeholder Detection

A description is a **placeholder** if any of these match:

1. Starts with an HTTP method + path: `^(GET|POST|PUT|DELETE|PATCH|HEAD|OPTIONS)\s+/`
2. Starts with `Module ` (apcore's default fallback)
3. Equals the function name in title case (e.g., `get_user` → `"Get User"`)
4. Equals the module_id with dots/underscores replaced by spaces
5. Length < 10 characters
6. Is empty or null

### Protocol Requirements Matrix

This matrix defines what each downstream consumer needs. Use it when deciding which fields to prioritize during refinement.

**MCP (Model Context Protocol) — LLM agent tool calling:**

| Field | How MCP Uses It |
|-------|----------------|
| `description` | Becomes the MCP Tool description — the LLM reads this to decide whether to call the tool |
| `input_schema` (full, with property descriptions) | Becomes the MCP Tool inputSchema — the LLM needs property descriptions to know what to pass |
| `metadata.x-when-to-use` | Appended to Tool description as "When To Use: ..." guidance |
| `metadata.x-when-not-to-use` | Appended to Tool description as "When Not To Use: ..." guidance |
| `metadata.x-common-mistakes` | Appended to Tool description as "Common Mistakes: ..." guidance |
| `annotations.readonly` | Mapped to `readOnlyHint` in MCP ToolAnnotations |
| `annotations.destructive` | Mapped to `destructiveHint` in MCP ToolAnnotations |
| `annotations.idempotent` | Mapped to `idempotentHint` in MCP ToolAnnotations |
| `annotations.requires_approval` | Mapped to `_meta.requiresApproval` — triggers human confirmation |
| `documentation` | Exposed as MCP Resource at `docs://{module_id}` — available for deep context |

**A2A (Agent-to-Agent) — AI agent skill discovery:**

| Field | How A2A Uses It |
|-------|----------------|
| `description` | Becomes AgentSkill description for skill discovery |
| `tags` | Used for AgentSkill filtering and categorization |
| `examples[].title` | Shown as AgentSkill example titles (max 10) |

**CLI (Command Line Interface) — developer invocation:**

| Field | How CLI Uses It |
|-------|----------------|
| `description` | Shown as command help text |
| `input_schema` (property descriptions) | Shown as `--flag` help text for each parameter |
| `annotations.requires_approval` | Triggers interactive HITL confirmation prompt |

### Annotation Inference Guide

When reading source code to infer annotations, look for these patterns:

| Annotation | Set to `true` when code... |
|-----------|---------------------------|
| `readonly` | Only reads data — no DB writes, no file writes, no external POST/PUT/DELETE calls |
| `destructive` | Deletes records, drops tables, removes files, revokes access, purges queues |
| `idempotent` | Uses PUT semantics, upsert patterns, or results are identical on repeat calls |
| `requires_approval` | Transfers money, deletes accounts, modifies permissions, sends bulk notifications |
| `open_world` | Makes HTTP calls, reads files, queries databases, runs subprocesses |
| `streaming` | Uses yield, async generators, SSE, or chunked transfer |
| `cacheable` | Pure function of inputs with no side effects — same inputs always produce same output |
| `paginated` | Returns partial results with cursor/offset/page tokens |
