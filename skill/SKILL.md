---
name: swivel-protocol
description: Context protocol for AI agents. Use when users mention "swivel", "context switching", "what were we working on", or want to set up explicit context management for multi-session work. Provides templates for .swivel.md files and teaches the swivel protocol.
---

# Swivel Protocol

Explicit context declaration for AI agents. One file per topic. Atomic context switches.

## The Problem

Every session: "What were we working on?" → Agent searches memory → Reconstructs state → Guesses wrong.

**Swivel solves this:** The `.swivel.md` file IS the context. No guessing.

## How It Works

1. **Root swivel** (`.swivel.md` in workspace root) — Tracks active/paused topics
2. **Topic swivels** (e.g., `trading/oakmont.swivel.md`) — Current state, files to load, continuity

## Protocol Rules (MANDATORY)

### Session Start

1. **Read `.swivel.md` FIRST** — Before any other action
2. **Load ONLY what's in `## Load`** — No extra files, no memory searches
3. **Report state** — "Working on X. Status: Y. Next: Z."

### Context Switch

When user says **"swivel to X"** or **"switch to X"**:

1. **STOP** — No referencing old topic
2. **READ** — Load X's `.swivel.md`
3. **UPDATE** — Move old topic to Paused, new topic to Active in root swivel
4. **CONFIRM** — "Switched to X. Status: Y. Next: Z."

### Session End

Before ending a session where you did work:

1. **Append to `## Continuity`** — Last action, key findings, next steps
2. **Update `## State`** — Current status, blockers, metrics
3. **Update `updated` timestamp** — In YAML frontmatter

## File Format

```markdown
---
swivel_version: 0.1.0
swivel_id: topic-name
swivel_type: topic|experiment|draft|spec
session_id: null
created: 2026-02-18T00:00:00Z
updated: 2026-02-18T00:00:00Z
updated_by: agent-name
mode: active|paused|completed|archived
---

## Load
- ./relevant-file.md
- ./src/main.py

## State
- **status**: in progress
- **started**: 2026-02-18T10:00:00Z
- **next**: specific next action

## Files
- **Main**: ./path/to/main.file
- **Config**: ./path/to/config.file

## Continuity
**Last Action** [timestamp]: What was done
**Key Finding** [timestamp]: Important discovery
**Next**: What happens next

## Blockers
- None OR specific blockers

## Decisions
[YYYY-MM-DD] Decision made and why
```

## Templates

### Root Swivel

See [assets/root-swivel.md](assets/root-swivel.md)

### Topic Swivel

See [assets/topic-swivel.md](assets/topic-swivel.md)

## Why This Works

1. **Explicit over implicit** — No reconstruction, no hallucination
2. **User controls context** — Edit the file, control what agents see
3. **Atomic switches** — Clean breaks between topics
4. **Token efficient** — Load 2KB of focused context, not 15KB of memory

## Common Patterns

### Starting New Work

```markdown
## Active Topics
- [new-project](./projects/new-project.swivel.md) ← **ACTIVE**

## Paused Topics
- [old-project](./projects/old-project.swivel.md)
```

### Pausing Work

```markdown
## Active Topics
- (none)

## Paused Topics
- [paused-project](./projects/paused-project.swivel.md)
```

### Multiple Active Topics

```markdown
## Active Topics
- [trading-bot](./trading/bot.swivel.md) ← **PRIMARY**
- [blog-post](./blog/post.swivel.md)
```

## Anti-Patterns

❌ Loading files not in `## Load`
❌ Searching memory for "what we were doing"
❌ Saying "if I recall correctly" after a context switch
❌ Forgetting to update `## Continuity` before session end

✅ Reading the swivel file and reporting exactly what's there
✅ Updating the swivel when work happens
✅ Using "swivel to X" as the canonical switch command