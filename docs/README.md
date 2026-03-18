# Documentation

## Structure

```
docs/
├── README.md              ← You are here
├── scope.md               ← What refinery does and does not do
├── getting-started.md     ← Installation and first run (TBD)
└── features/
    ├── overview.md        ← Feature summary (TBD)
    ├── scoring.md         ← Quality scoring model and rubric (TBD)
    ├── refinement.md      ← LLM-powered metadata enhancement (TBD)
    ├── source-resolver.md ← Multi-language source code resolution (TBD)
    └── ci-integration.md  ← CI/CD quality gates (TBD)
```

## Reading Order

1. **[Scope & Boundaries](scope.md)** — Start here to understand what refinery is and isn't.
2. **[Idea Draft](../ideas/draft.md)** — Problem validation and initial requirements.
3. Feature docs (once available) for implementation details.

## Concept Index

| Concept | Authoritative Definition |
|---------|------------------------|
| QualityModel | [scope.md § Quality Model](scope.md#quality-model-what-good-metadata-means) |
| ScannedModule | [apcore-toolkit docs](https://github.com/aipartnerup/apcore-toolkit) |
| .binding.yaml | [apcore-toolkit: Output Writers](https://github.com/aipartnerup/apcore-toolkit/blob/main/docs/features/output-writers.md) |
| MCP Tool Definition | [apcore-mcp](https://github.com/aipartnerup/apcore-mcp-python) |
| A2A AgentSkill | [apcore-a2a](https://github.com/aipartnerup/apcore-a2a-python) |
