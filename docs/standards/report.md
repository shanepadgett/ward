---
match:
  - internal/finding/**
  - internal/report/**
  - internal/output/**
  - internal/gate/**
---

# Standard — Report & findings

How to shape findings, reports, and gate inputs.

## Principle

Gate logic and machine consumers use a **small normalized spine**. Drivers map into it. Raw upstream tool JSON is not the public contract.

## Finding spine

Every finding carries at least:

| Field | Rule |
|---|---|
| `tool` | Upstream tool id |
| `tool_version` | When known; omit or empty only if unavailable |
| `rule_id` | Upstream rule / diagnostic code |
| `severity` | Shared enum only (define once; do not invent per-tool strings) |
| `path` | Repo-relative |
| span | start/end line/col when upstream provides them |
| `message` | Clear text; do not require consumers to parse it for control flow |
| `fix_available` | bool |
| `fingerprint` | Stable enough for baseline/diff de-dupe |
| `category` | Coarse shared bucket, not free-form per call site |

Do not expose every upstream rule as a Ward CLI flag. Gate on the spine.

## Report document

- Include a `schema_version`. Incompatible shape changes bump it.
- `ok` means gate success, not “process started.”
- Summarize counts plus `tools_used` and `tools_skipped`.
- Every skip has a **reason**.
- Empty collections encode as `[]`, not `null`.
- Hard failures may add a structured `error` (`kind`, `message`, `hint`) separate from gate-outcome findings.

## Stability

- Removing or redefining a spine field is breaking once shipped.
- Additive optional fields are fine if unknown fields can be ignored.
- Fingerprint algorithm changes break baselines — treat as breaking and document migration.

## Gate

- Diff/baseline uses spine fields (path, fingerprint, severity, …), not raw tool payloads.
- Policy failure still writes the report to stdout. Do not put the only copy on stderr.
