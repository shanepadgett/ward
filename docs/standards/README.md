# Scoped standards

How-to-work rules for specific trees. Not product inventories.

Each `*.md` here (except this file) has YAML front matter:

```yaml
---
match:
  - internal/example/**
---
```

| File | Work it governs |
|---|---|
| [cli.md](./cli.md) | Command layer, streams, flags, exits, help |
| [driver.md](./driver.md) | Stack drivers |
| [report.md](./report.md) | Findings, report shape, gate inputs |

Nudge extension: soft nudge on read of a matched path; hard nudge on edit unless that standard is already in context. Never blocks. Never inlines the file.
