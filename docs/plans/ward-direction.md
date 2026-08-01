# Ward direction

Working notes from early scoping. Not a PRD.

## Goal

Gate agent-written code. One entry point that runs the right existing tools, normalizes output, and fails on new junk introduced in a change. Name means keep bad agent output out.

Reference we studied: [fallow](https://github.com/fallow-rs/fallow) — deep Rust TS/JS graph analyzer. Useful for pipeline ideas (diff-scoped audit, stable JSON, exit codes). Not what we're rebuilding.

## What we're building

Orchestrator + agent contract, not a new multi-language compiler.

```
detect stack → run driver tools → normalize → diff/baseline gate → JSON + exit codes
```

Optional: ward-native checks tools don't cover (secrets in diff, forbidden paths, "no tests touched", etc.).

Custom deep analysis only when a real gap hurts and no good tool exists.

## What we're not building (for now)

- Fallow-class in-process graph engine across languages
- Tree-sitter-as-universal-brain
- Equal depth in every language
- Re-implementing ESLint/Clippy/Roslyn rules inside ward
- Bundling full language SDKs

## Drivers

A **driver** is the pluggable unit for a stack (e.g. TS, Go). It owns the lifecycle for that stack:

1. Detect whether the stack applies
2. Resolve binaries (local → PATH → skip/fail)
3. Invoke tools with machine-readable flags
4. Parse upstream output (SARIF/JSON)
5. Map into the shared finding spine
6. Report what ran and what was skipped

Not called "packs" — empty word. Not "adapters" for the unit as a whole — adapting shapes is one step; the job is operating a foreign toolchain end to end. Per-tool argv/parse helpers can live inside a driver without being a separate product concept.

## Languages

Interest list: TypeScript/JavaScript, C#, Go, Rust, Java, Kotlin, Swift.

Ship shape: multi-language *ready* (shared finding schema + driver interface), one driver deep at a time.

1. TS/JS first (knip, oxlint/eslint, tsc, formatter, etc.)
2. Go second if we want to prove the IR/driver model (clean toolchain)
3. Rest as drivers when needed — honest skips when a capability doesn't exist

## Tool ownership

Prefer the repo's binaries and configs (version matches their setup).

Resolve order per check:

1. Explicit config override
2. Project-local (`pnpm exec`, package bins, go/cargo tools)
3. PATH
4. Optional ward-managed pin (only when the driver declares it safe)
5. Skip or fail per policy — never silent fake pass

Ward config owns orchestration and gate policy. Foreign rule config stays in foreign files.

## Version / output pain

Drivers deal with invoke flags + machine-readable output (prefer SARIF/JSON). Support ranges per driver major. Normalize a small spine only:

`tool`, `tool_version`, `rule_id`, `severity`, `path`, span, `message`, `fix_available`, fingerprint, category

Gate logic sits on that spine. Don't expose every upstream rule as a ward flag.

## Shipping / consumption

- Go CLI, single static binary (good fit for exec, JSON, cross-compile)
- Later: Action wrapper, agent skill/MCP if useful
- `mise.toml` already pins Go for this repo

Out-of-box means ward runs, detects what it can, reports `tools_used` / `tools_skipped`. Not "all seven SDKs included."

TS driver can get closest to batteries-included via downloadable Node tools. Go/Rust/C# expect their normal toolchains on the machine.

## Commands (sketch)

| Command | Role |
|---|---|
| `ward guard` | Soul: changed-work gate for agents/CI |
| `ward check` | Broader full/driver run |
| `ward init` | Non-destructive scaffold: `ward.toml`, missing tool defaults, pins |
| `ward init --recommend` | Plan only |
| `ward init --force` | Overwrite ward-managed paths only |
| `ward fix` | Later; delegate to tools that autofix |

Init does not run from guard. Overwrite is opt-in and narrow.

## Defaults to decide later

- Missing tool: fail closed vs best-effort skip (product default matters)
- Exact TS v1 tool list and profiles (`agent`, `pr`, `strict`)
- Finding schema v1 and exit code contract
- How aggressive init recommended configs are

## Implementation language

Go for core. Drivers are thin Go modules (detect, resolve, argv, parsers). Heavy work stays upstream.

Suggested layout vocabulary: `internal/driver`, stack impls under e.g. `internal/driver/ts`, config as `[[drivers]]` — not packs.

## Standards

- Always-on (Go baseline, vocabulary): `AGENTS.md`
- Scoped (CLI / driver / report): `docs/standards/*.md` with `match` globs in YAML front matter
- Intended for a pi standards-nudge hook: soft nudge on matched reads (read standard only if editing), hard nudge on edit unless that standard is already in context. Extension does not inline standard bodies. See `docs/standards/README.md`.
