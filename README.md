---
swivel_version: 0.2.0
swivel_id: swivel-protocol
swivel_type: spec
---

# Swivel — Session Context Protocol

**Trust, but verify.** Explicit context declaration with reality validation for AI agents.

---

## The Problem

Every AI agent session starts the same way:

- **User:** "Hey, can you check on that thing?"
- **Agent:** "What thing?"
- **User:** "The thing from yesterday."
- **Agent:** *searches memory, reconstructs state, guesses wrong*

Swivel replaces reconstruction with explicit context loading. A `.swivel.md` file tells an agent exactly what state to load, what files to read, and what's current — with verification that it matches reality.

---

## Quick Start

Add Swivel to any AI agent in 2 minutes. It's just markdown.

### 1. Create a `.swivel.md` file

```markdown
---
swivel_version: 0.2.0
swivel_id: my-project
swivel_type: topic
created: 2026-02-24T00:00:00Z
updated: 2026-02-24T00:00:00Z
mode: active
ttl_hours: 24
---

## Load
- ./my-project.ts
- ./README.md

## State
- **status**: in progress
- **current task**: Refactoring auth module

## Continuity
**Last:** Set up project structure
**Next:** Implement login flow
```

### 2. Tell your agent to read it

**Claude Code:**
```
Read .swivel.md first, then help me with this project.
```

**Cursor:**
Add to your prompt: "Always read `.swivel.md` before responding."

**Any AI agent:**
```
Before helping with this project, read the .swivel.md file 
in the root directory for current context.
```

### 3. Done

Your agent now loads explicit context every session. No reconstruction. No "what were we working on?"

---

## Why Swivel?

| | Swivel | .cursorrules | CLAUDE.md | System Prompts | mem0 context |
|---|---|---|---|---|---|
| **TTL / Auto-stale** | ✅ Yes | ❌ No | ❌ No | ❌ No | ⚠️ Partial |
| **Drift detection** | ✅ Built-in | ❌ No | ❌ No | ❌ No | ❌ No |
| **Healthchecks** | ✅ Command/file/http | ❌ No | ❌ No | ❌ No | ❌ No |
| **Verified state** | ✅ `last_verified` timestamp | ❌ No | ❌ No | ❌ No | ❌ No |
| **Standard format** | ✅ Portable across agents | ⚠️ Cursor-only | ⚠️ Claude-only | ⚠️ Per-platform | ⚠️ Proprietary |
| **Human readable** | ✅ Pure markdown | ✅ Yes | ✅ Yes | ❌ Hidden | ❌ Opaque |

Swivel is different because it acknowledges a hard truth: **context drifts**. Rules files go stale. Memory systems hallucinate. Swivel adds TTL, healthchecks, and drift detection so agents know when to re-verify instead of trust.

---

## How It Works

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│   .swivel.md    │────▶│  Agent reads │────▶│  Loads explicit │
│  (context file) │     │   on startup │     │   files only    │
└─────────────────┘     └──────────────┘     └─────────────────┘
         │                                              │
         ▼                                              ▼
┌─────────────────┐                          ┌─────────────────┐
│  Healthcheck    │◄─────────────────────────│  Work happens   │
│  (reality check)│                          │  (session)      │
└─────────────────┘                          └─────────────────┘
         │                                              │
         ▼                                              ▼
┌─────────────────┐                          ┌─────────────────┐
│  Drift detected?│─────────────────────────▶│  Update swivel  │
│  (TTL/process)  │                          │  before ending  │
└─────────────────┘                          └─────────────────┘
```

---

## Key Features

- **TTL (time-to-live)** — Auto-expire after N hours. Forces re-verification.
- **Healthcheck integration** — Link to processes, files, or HTTP endpoints
- **Drift taxonomy** — `ttl_expired`, `process_mismatch`, `file_missing`, `topic_orphan`, `handoff_failed`, `blocker_stale`
- **Portable** — Works with Claude, Cursor, GitHub Copilot, or any agent that reads files
- **Human-readable** — Pure markdown. No proprietary formats.

---

## Full Specification

See [SPEC.md](./SPEC.md) for the complete v0.2 protocol specification, including:
- Complete file format reference
- Drift taxonomy and handling
- Root swivel structure
- CLI tooling
- Migration guide from v0.1

---

## SwivelOS Ecosystem

- **[swivel-protocol](https://github.com/SwivelOS/swivel-protocol)** — This repo. The context protocol spec.
- **[swivcast](https://github.com/SwivelOS/swivcast)** — AI agent podcast framework. Turn multi-agent conversations into podcast episodes.
- **[recall](https://github.com/SwivelOS/recall)** — Experiential memory system for AI agents. Query past sessions, actions, and outcomes.

---

*Trust, but verify.*
