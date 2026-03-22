# AGENTS.md

Guidance for AI coding agents working on the Multi-Agent Spec repository. This file helps agents understand the project structure, conventions, and development workflow.

## Project Overview

Multi-Agent Spec is a platform-agnostic specification for defining multi-agent AI systems. It provides:

- **Agent definitions** — Markdown + YAML frontmatter in `specs/agents/` or `examples/*/agents/`
- **Team definitions** — JSON in `specs/teams/` or `examples/*/team.json`
- **Deployment configs** — JSON in `specs/deployments/` or `examples/*/deployment.json`
- **SDKs** — Go (`sdk/go/`), Python (`sdk/python/`), TypeScript (`sdk/typescript/`)
- **Schemas** — JSON Schema in `schema/` (source of truth for validation)

## Directory Structure

```
├── schema/              # JSON Schema definitions (agent, team, deployment)
├── sdk/
│   ├── go/              # Go types and loader
│   ├── python/          # Pydantic models
│   └── typescript/      # Zod schemas (generated from JSON Schema)
├── tools/generate/      # Codegen: Go types → JSON Schema → TypeScript
├── examples/            # Example agent teams (e.g. stats-agent-team)
└── .github/workflows/   # CI, lint, SAST
```

## Build and Test Commands

### All-in-one

```bash
make all          # generate + build + test
make install      # install dependencies
```

### Codegen pipeline

```bash
make generate             # full pipeline
make generate-schema      # Go → JSON Schema only
make generate-typescript  # JSON Schema → TypeScript/Zod
```

**Order matters**: Schema changes should flow Go → JSON Schema → TypeScript. Do not hand-edit `sdk/typescript/src/generated/` — it is generated.

### Tests

```bash
make test                 # Go + TypeScript
make test-go              # cd sdk/go && go test ./...
make test-typescript      # cd sdk/typescript && npm test
```

Python tests (run from `sdk/python/`):

```bash
cd sdk/python && pytest -v
```

### Linting

```bash
make lint-schema          # schemago lint (if installed)
cd sdk/go && golangci-lint run
cd sdk/typescript && npm run lint && npm run typecheck
cd sdk/python && ruff check . && mypy .
```

## Schema Conventions

When changing JSON schemas (`schema/agent/`, `schema/orchestration/`, `schema/deployment/`):

1. **Go compatibility** — Avoid `anyOf`/`oneOf` without discriminators; add `const` discriminator fields for unions.
2. **Validation** — Run `schemago lint` on schema files when possible.
3. **Regenerate** — After schema edits, run `make generate` to update TypeScript and keep Go types in sync.
4. See README "JSON Schema Guidelines for Go Compatibility" for details.

## Agent Definition Format

Agent definitions use Hugo-compatible Markdown with YAML frontmatter:

```markdown
---
name: my-agent
description: Brief description
model: sonnet        # haiku | sonnet | opus
tools: [WebSearch, Read, Write]
---

Instructions as markdown...
```

- Store in `specs/agents/` or `examples/<team>/agents/`
- Subdirectories become namespaces (e.g. `prd/lead` from `agents/prd/lead.md`)
- Referenced by name in team JSON: `"agents": ["orchestrator", "prd/lead"]`

## Development Environment

- **Go** — 1.24+ (see `.github/workflows/ci.yaml`)
- **Node** — 18+ for TypeScript SDK
- **Python** — 3.10+ for Python SDK
- **Optional** — `schemago` for schema linting (`go install github.com/grokify/schemago/cmd/schemago@latest`)

## PR and Commit Guidelines

- Use clear, descriptive commit messages.
- Prefer smaller, focused commits.
- Run `make test` and relevant lint commands before committing.
- PR titles: descriptive of scope (e.g. `Add AGENTS.md for AI coding agents`).
- Do not push to branches other than the designated feature branch unless instructed.

## File Boundaries

| Path | Purpose | Editable by hand? |
|------|---------|-------------------|
| `schema/**/*.json` | JSON Schema definitions | Yes |
| `sdk/go/*.go` | Go types, loader, mappings | Yes |
| `tools/generate/*.go` | Codegen tooling | Yes |
| `sdk/typescript/src/generated/` | Generated Zod/TS from schema | No — regenerate |
| `sdk/python/src/` | Pydantic models | Yes |
| `examples/*/agents/*.md` | Agent definitions | Yes |

## Related Documentation

- [README.md](README.md) — Full spec, schemas, platforms, tool mappings
- [schema/](schema/) — JSON Schema files
- [examples/stats-agent-team/](examples/stats-agent-team/) — Reference implementation
