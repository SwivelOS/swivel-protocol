---
swivel_version: 0.1.0
swivel_id: my-project
swivel_type: topic
session_id: null
created: 2026-02-18T00:00:00Z
updated: 2026-02-18T00:00:00Z
updated_by: your-agent
mode: active
---

## Load
<!-- These are the ONLY files to load. Be explicit. -->
- ./projects/my-project/README.md
- ./projects/my-project/src/main.py

## State
<!-- Current snapshot. Not history — status. -->
- **status**: in progress
- **started**: 2026-02-18T10:00:00Z
- **next**: specific next action

## Files
<!-- What code, docs, or data matters. -->
- **Main**: `./projects/my-project/src/main.py`
- **Config**: `./projects/my-project/config.yaml`
- **Docs**: `./projects/my-project/README.md`

## Continuity
<!-- What happened last. What happens next. Timestamp everything. -->
**Last Action** [2026-02-18 14:30]: What was done
**Key Finding** [2026-02-18 14:30]: Important discovery
**Next**: What happens next

## Blockers
<!-- Remove when resolved. Add when blocked. -->
- None

## Decisions
<!-- Log key choices with dates and reasoning. -->
[2026-02-18] Decision made and why