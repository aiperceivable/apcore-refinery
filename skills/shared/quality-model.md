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

### Silent Degradation Detection

Scanners across all frameworks silently fall back to empty schemas when extraction fails. These patterns indicate **degraded metadata**, not "no parameters":

| Pattern | What It Means | Likely Cause |
|---------|--------------|-------------|
| `input_schema: {type: object, properties: {}}` | Schema extraction failed silently | Flask: missing Pydantic/marshmallow model. Django: $ref resolution failure. NestJS: @ApTool without inputSchema. |
| `description` == `documentation` (identical text) | Documentation is just a copy of description | Scanner didn't find distinct docstring body |
| `additionalProperties: true` on input_schema | Permissive fallback | TipTap: unknown extension command. NestJS: DTO extraction failure. |
| All annotations at defaults despite HTTP method context | Annotation inference didn't run or was overridden | Scanner bug or manual override |

### Protocol-Specific Formatting Rules

These rules encode how the apcore bridges actually transform metadata. Following them ensures the refined YAML produces optimal results at each consumer.

#### MCP Tool Description Assembly

The MCP bridge **appends** intent metadata to the description. The final Tool description an LLM sees is:

```
{description}

When To Use: {metadata.x-when-to-use}
When Not To Use: {metadata.x-when-not-to-use}
Common Mistakes: {metadata.x-common-mistakes}
Workflow Hints: {metadata.x-workflow-hints}
```

This means:
- **description** should be a standalone, complete sentence — it's the first thing the LLM reads
- **x-when-to-use** should NOT repeat the description; it should add decision-making context (e.g., "When the user wants to add a new item" not "Use this tool to create a new task")
- **x-when-not-to-use** should reference **specific alternative modules by name** (e.g., "For updating existing tasks, use update_task.put instead")
- **x-common-mistakes** should describe **caller errors**, not user errors (e.g., "Omitting the required title field" not "Users forget to fill in the title")

#### MCP Input Schema: x-llm-description

The MCP bridge preserves `x-llm-description` fields inside input_schema properties. In OpenAI strict mode, `x-llm-description` gets **promoted** to become the property's `description`, then all other x-* fields are stripped. This means:

- Add `x-llm-description` to input_schema properties when the regular `description` is insufficient for LLM parameter understanding
- Keep `x-llm-description` under 200 characters (CLI truncates at 200 chars when using it as --flag help text)
- Write it as an instruction to the LLM: "The unique integer ID of the task to delete. Use list_tasks.get to find valid IDs."

#### A2A: Example Titles Must Be Self-Descriptive

The A2A bridge extracts **only example titles** (max 10), discarding inputs and outputs. This means example titles must convey the use case by themselves:

- Bad: `"Basic example"`, `"Example 1"`, `"Test case"`
- Good: `"Create admin user with full permissions"`, `"Delete completed task by ID"`

#### A2A: Annotations Are Lost — Embed Safety in Description

Due to a known issue in the A2A bridge (`_build_extensions()` is defined but never called), **all annotation metadata is lost** for A2A consumers. A2A agents have NO knowledge of whether a skill is destructive or requires approval.

For destructive or approval-requiring modules, embed a safety warning directly in the description:
- `"Permanently delete a task by its ID (destructive, irreversible)"`
- `"Transfer funds between accounts (requires human approval)"`

#### CLI: File Parameter Convention

The CLI bridge treats input_schema properties as --flags. Properties ending with `_file` or having `x-cli-file: true` become `click.Path(exists=True)` file inputs. When a parameter accepts a file path, add `x-cli-file: true` to its schema.

### Framework-Specific Quality Patterns

When refining, detect the source framework (from `metadata.source` or project config) and apply framework-specific knowledge:

**Flask (flask-apcore):**
- Multi-backend schema chain: Pydantic > marshmallow > type hints > empty
- Empty schema often means no Pydantic/marshmallow model on the route — check for `request.json` or `request.args` patterns in the source code
- Blueprint structure maps to tags

**Django (django-apcore):**
- DRF ViewSets: custom `@action` methods often lack description/summary
- django-ninja: generates operationId but not description by default
- Complex nested $ref schemas may be partially resolved

**NestJS (nestjs-apcore):**
- @ApTool decorator is opt-in; undecorated methods are invisible to scanning
- DTOs need class-validator decorators for proper schema extraction
- Schema extraction failures fall back to empty schema **silently** (only logged)
- JSDoc is optional; if missing, description comes solely from decorator

**TipTap (tiptap-apcore):**
- 79 built-in commands with hand-crafted schemas; custom extension commands get permissive defaults
- Unknown commands use `additionalProperties: true` and DEFAULT_ANNOTATIONS (all false)
- Auto-generated descriptions from command names (camelCase → "toggle bold") lack semantic depth

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
