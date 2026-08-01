---
match:
  - cmd/**
  - internal/cli/**
  - internal/cmd/**
---

# Standard — CLI

How to change command surface, flags, help, and process I/O.

## Layering

- Command layer: parse flags/args → typed config → call `internal/...`.
- Domain packages do not import the CLI framework (Cobra, etc.).
- `RunE` (or equivalent) returns `error`. `os.Exit` only in `main`.
- Config merge order: flag > env (`WARD_*`) > `ward.toml` > default.

## Streams

- stdout = primary data only (human report or machine document).
- stderr = progress, diagnostics, human-facing errors.
- Never mix progress or logs into machine stdout.
- No spinners or cursor rewrites when the stream is not a TTY.

## Output format

- One global format flag; stick to it (`--format` preferred).
- Values at least: `auto`, `text`, `json`. `--json` may alias `json`.
- Explicit format always wins over TTY detection.
- Default posture: text on TTY; JSON when piped or forced (agents/CI).
- No ANSI when not a TTY. Honor `NO_COLOR` and `TERM=dumb`. Support `--no-color`.

## Exit codes

Treat exit codes as a stable API. Scripts and agents branch on `$?` without parsing text.

| Code | Meaning |
| --- | --- |
| 0 | Success / gate passed |
| 1 | Gate failed (findings over policy) — report still on stdout |
| 2 | Usage / bad flags |
| 3 | Config invalid |
| 4 | Required tool missing (fail-closed) |
| 5 | Internal unexpected failure |

- Code 1 is an **outcome**, not a crash. Do not overload it for panics or IO failures.
- Changing the meaning of an assigned code is a breaking change.

## Flags

- Prefer flags over multi-meaning positionals.
- Long form for anything scripts use. Short only for high-frequency human flags.
- Same flag name means the same thing on every command.
- Removing or renaming a released flag is breaking; prefer additive + deprecate.

## Interactivity

- Core automation commands must run with no prompts.
- No TTY: never prompt. Destructive actions without an explicit bypass flag must refuse, not assume yes.
- Secrets never on argv. Use env, file, or stdin.

## Help

- Lead with examples on non-trivial commands.
- Document exit codes where operators need them.
- Print usage on bad flags/args; do not dump full usage on every runtime error.
