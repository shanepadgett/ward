---
match:
  - internal/driver/**
  - internal/drivers/**
---

# Standard — Drivers

How to implement or change a stack driver.

## Scope of a driver

A driver operates one stack end to end:

1. Detect whether the stack applies
2. Resolve tool binaries
3. Invoke with cancelable context
4. Parse machine-readable upstream output
5. Normalize into the shared finding spine
6. Account for tools used and skipped (with reasons)

Stay thin. Heavy analysis stays in upstream tools.

## Naming

- Call the unit a **driver**. Not pack. Not adapter-for-the-whole-unit.
- Per-tool argv/parse helpers inside a driver are fine; they are not a separate product type.

## Binary resolve order

Per check, in order:

1. Explicit config override
2. Project-local (package bins, `pnpm exec`, language toolchains)
3. PATH
4. Ward-managed pin only if this driver opts in as safe
5. Skip or fail per policy

Never report success for a check that did not run.

Foreign rule config stays in foreign files. Ward config orchestrates; it does not re-host upstream rule trees.

## Invoke and parse

- Prefer upstream SARIF/JSON (or other stable machine formats).
- Honor context cancel and timeouts; no orphaned tool processes.
- Record tool identity and version when available.
- Map into the finding spine only — do not leak raw upstream schemas into gate logic or the public report as the source of truth.

## Skips vs errors vs findings

- Optional tool missing → skip + reason in the report.
- Required tool missing under fail-closed policy → hard failure (not an empty pass).
- Upstream non-zero with parseable diagnostics → findings.
- Upstream crash or unparseable output → explicit error; never invent empty success.

## Tests

- Fixture upstream payloads → golden normalized findings.
- Cover resolve order and skip reasons.
- Cover cancel/timeout when a tool hangs.
