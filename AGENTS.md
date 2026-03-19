# AGENTS.md

## Cursor Cloud specific instructions

This is a specification + SDK library project (Go, Python, TypeScript) with no running services. The `Makefile` orchestrates the full codegen + build + test pipeline.

### Quick reference

| Task | Command |
|---|---|
| Full pipeline (generate + build + test) | `make all` |
| Go SDK tests | `cd sdk/go && go test ./...` |
| TypeScript SDK tests | `cd sdk/typescript && npm test` |
| Python SDK tests | `cd sdk/python && PYTHONPATH=src pytest tests/ -v` |
| Go lint | `cd sdk/go && golangci-lint run` |
| Python lint | `cd sdk/python && ruff check src/ tests/` |
| Python type-check | `cd sdk/python && PYTHONPATH=src mypy src/` |
| TypeScript build | `cd sdk/typescript && npm run build` |
| Schema generation | `make generate` |

### Non-obvious caveats

- **Go 1.24+ required**: `tools/generate/go.mod` requires Go 1.24. The VM ships with Go 1.22; the update script installs 1.24.1 to `/usr/local/go`. Ensure `/usr/local/go/bin` is on `PATH`.
- **Python SDK PYTHONPATH**: The `sdk/python/pyproject.toml` references a `readme = "README.md"` that does not exist in `sdk/python/`, so `pip install -e .` fails. Set `PYTHONPATH=/workspace/sdk/python/src` when running Python tests or mypy.
- **Python lint baseline**: `ruff check` reports 4 pre-existing issues (3 import sorting, 1 line length). `mypy --strict` reports 1 pre-existing error (missing generic type param in `models.py:250`). These are not regressions.
- **Codegen pipeline direction**: Go types (`sdk/go`) → JSON Schema (`schema/`) → TypeScript/Zod (`sdk/typescript`). Python SDK models are hand-written.
- **golangci-lint**: Installed to `/usr/local/bin`. The CI uses `golangci-lint-action` with `working-directory: sdk/go`.
