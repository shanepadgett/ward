# Agent notes — Ward

## What this is

Ward gates agent-written code. Go CLI orchestrates existing tools, normalizes findings, fails on new junk in a change. Not a multi-language analyzer.

## Go

### Format and lint

- All Go is `gofmt` / `goimports`.
- Prefer `staticcheck` (via golangci-lint when configured). Fix what the tools flag.
- Quality gate: `mise run check` (format → vet → staticcheck → test → markdown). Quiet, sequential, path:line tool output.
- Atomic tasks: `format`, `format:fix`, `format:check`, `vet`, `lint`, `test`, `build`, `tidy`, `markdown`.

### Layout

- Single binary: `cmd/ward` thin `main` (wire + `os.Exit` once).
- Real code under `internal/`. Nothing external should import us until we deliberately publish a module API.
- No `pkg/` until something is actually consumed outside this module.
- No grab-bag packages: `util`, `common`, `helpers`, `misc`.
- Name packages by domain: `driver`, `finding`, `gate`, `config`, …

### Code habits

- Errors: check them; wrap with `%w`; strings lowercase, no trailing period.
- Early return; happy path left-aligned. Don't panic for routine failure.
- `context.Context` is the first argument on anything that runs tools, does I/O, or should cancel.
- Interfaces live with the consumer. Implementors return concrete types.
- Short names; initialisms keep consistent case (`URL`, `ID`, `HTTP`).
- Table-driven tests next to the code. Failure messages show got/want and inputs.
- Prefer synchronous APIs. If you start goroutines, make exit obvious (esp. tool runners).
- JSON that leaves the process: empty slices as `[]`, not `null`.

## Vocabulary

- **Driver** — pluggable unit for a stack (TS, Go, …). Owns detect → resolve → run → normalize → skip reporting.
- Not "pack". Not "adapter" for the whole unit (normalize is one step inside the driver).

## Don't

- Reimplement upstream linters inside Ward.
- Silent success when a required tool didn't run.
- Expand scope past what the user asked.
