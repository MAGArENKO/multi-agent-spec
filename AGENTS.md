# AGENTS.md

## Cursor Cloud specific instructions

This is an SDK/schema monorepo with three SDKs (Go, TypeScript, Python) and a codegen tool. There are no running services, databases, or containers.

### Project structure

| Component | Path | Language | Purpose |
|---|---|---|---|
| Go SDK | `sdk/go/` | Go ≥1.24 | Types, loader, validator, renderer |
| TypeScript SDK | `sdk/typescript/` | Node ≥18 + npm | Zod schemas and types (generated from JSON schemas) |
| Python SDK | `sdk/python/` | Python ≥3.10 | Pydantic models |
| Code generation | `tools/generate/` | Go | Generates JSON Schemas from Go types |

### Key commands

All orchestrated via the root `Makefile`. See `make help` for the full list.

- **Full pipeline:** `make all` (generate → build → test)
- **Codegen:** `make generate` (Go types → JSON Schema → TypeScript/Zod)
- **Build:** `make build` (TypeScript SDK compilation)
- **Tests:** `make test` (Go + TypeScript), or `make test-go` / `make test-typescript` individually
- **Python tests:** `cd sdk/python && PYTHONPATH=src:$PYTHONPATH pytest tests/ -v`
- **Lint:** `go vet ./...` (Go), `npx tsc --noEmit` (TypeScript), `ruff check src/ tests/` (Python)
- **Install deps:** `make install`

### Gotchas

- **Go ≥1.24 required:** `tools/generate/go.mod` specifies `go 1.24.0`. The default system Go may be older; the update script installs Go 1.24.1 to `/usr/local/go`.
- **Python SDK missing README.md:** `sdk/python/pyproject.toml` declares `readme = "README.md"` but the file does not exist. `pip install -e ".[dev]"` will fail. Install deps directly instead: `pip install pydantic "pytest>=8.0.0" "pytest-cov>=4.0.0" "ruff>=0.4.0" "mypy>=1.10.0"`. Then run tests with `PYTHONPATH=src:$PYTHONPATH`.
- **Python ruff pre-existing warnings:** The Python SDK has import-ordering lint warnings (I001) in existing code. These are not regressions.
- **PATH:** Ensure `/usr/local/go/bin` and `~/.local/bin` are on PATH for Go and Python CLI tools.
