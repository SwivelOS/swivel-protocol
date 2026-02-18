# Swivel — Context Protocol for AI Agents

> **Swivel to what matters. The file knows.**

Stop asking "what were we working on?" Swivel gives AI agents explicit context declaration — one file per topic, atomic context switches, no more reconstruction.

## Quickstart

**Start using Swivel in 60 seconds:**

1. **Create `.swivel.md` in your workspace root:**
   ```markdown
   ## Active Topics
   - [my-project](./my-project.swivel.md)
   
   ## Paused Topics
   (none yet)
   ```

2. **Create a topic swivel for your current work:**
   ```markdown
   ## Load
   - ./README.md
   - ./src/main.py
   
   ## State
   - **status**: in progress
   - **next**: finish authentication module
   
   ## Continuity
   **Last**: Set up project structure
   **Next**: Implement OAuth flow
   ```

3. **Tell your agent:**
   > "Read `.swivel.md` first every session. Load only what's in `## Load`."

**That's it.** Your agent now has explicit context, not reconstructed guesses.

## The Problem

Every AI agent session starts the same way:

- **User:** "Hey, can you check on that thing?"
- **Agent:** "What thing?"  
- **User:** "The thing from yesterday. The trading bot."
- **Agent:** *searches memory, reconstructs state, guesses wrong*

This wastes tokens, introduces hallucination, and kills momentum.

**Swivel solves this.** One file per topic. Explicit load order. Context switches become atomic.

## Why Not Just...

**Why not just MEMORY.md?**
Memory files accumulate everything. They grow unbounded. A swivel is point-in-time — only what matters NOW for THIS topic.

**Why not just chat history?**
Chat history is conversation, not state. It's linear, not structured. Agents can't "jump to the current status" without reading everything.

**Why not just git?**
Git is code history. Swivel is agent context. They serve different purposes. Git tells you what changed; swivel tells you what matters.

**Why not just prompt the agent with context?**
You can, but you'll repeat yourself every session. Swivel makes it permanent — update the file once, every session gets it.

## Core Concepts

### Swivel File
A `.swivel.md` file is a context declaration. It tells an agent:
- What state to load
- What files to read
- What the current status is
- Where to find continuity

### Root Swivel
The `.swivel.md` in your workspace root tracks:
- **Active topics** — what you're working on NOW
- **Paused topics** — not abandoned, just parked
- **Global ignore** — patterns to never load

### Topic Swivels
Individual `.swivel.md` files in subdirectories track specific work streams:
- `trading/oakmont-ab-test.swivel.md`
- `prism/packaging.swivel.md`
- `blog/post-2026-02-18.swivel.md`

## File Format

```markdown
---
swivel_version: 0.1.0
swivel_id: oakmont-ab-test
swivel_type: topic|experiment|draft|spec
session_id: abc123-...-xyz    # null if not currently active
created: 2026-02-17T22:00:00Z
updated: 2026-02-17T22:30:00Z
updated_by: swiv
mode: active|paused|completed|archived
---

## Load
These are the ONLY files to load. Be explicit.
- ./oakmont-ab-test.swivel.md
- ./memory/2026-02-17.md
- ../SOUL.md
- ../USER.md

## State
Current snapshot. Not history — status.
- **status**: running
- **mode**: paper trading
- **started**: 2026-02-17T22:02:00Z
- **key_metric**: 8¢ improvement on entry prices
- **next_checkpoint**: tomorrow AM

## Files
What code, docs, or data matters.
- **Control code**: `/trading/oakmont/src/scanner.ts`
- **Variant code**: `/trading/oakmont-v2/src/scanner.ts:138`
- **Comparison script**: `/trading/oakmont/compare-versions.ts`

## Continuity
What happened last. What happens next.
**Last Action** [22:06]: Started parallel A/B test, resolved DuckDB locking
**Key Finding** [22:06]: v2.0-settle averaging 35.9¢ entry vs v1.0's 43.8¢
**Next Expected** [tomorrow AM]: Stop bots, run comparison, analyze P&L

## Blockers
- None

## Decisions
[2026-02-17] Use separate DuckDB files (locking conflict resolution)
[2026-02-17] 45s settling period (allows price discovery)
```

## Protocol Rules

### For Agents (MANDATORY)

**1. Read `.swivel.md` first**
```
Every new session MUST read `./.swivel.md` before any other action.
```

**2. Load ONLY what's listed**
```
The `## Load` section is the complete list. Don't load files not listed.
Don't search memory for "what we were doing." The swivel IS the source of truth.
```

**3. Report what you see**
```
When asked "what are we working on?", read the swivel and report:
"Oakmont A/B test. Status: running. 11 trades on v2.0, 4 on v1.0.
Next checkpoint: tomorrow AM."
```

**4. Drop context on switch**
```
When user says "swivel to X", "switch to X", or "let's work on Y":
1. STOP referencing old topic
2. READ the new topic's .swivel.md
3. UPDATE root .swivel.md (move old to Paused, add new to Active)
4. CONFIRM: "Switched to [topic]. [Current state from .swivel.md]"

"Swivel to X" is the canonical phrase. It's short, memorable, and names the protocol.
```

**5. Update before ending**
```
Before ending a session where you did work:
- Append to `## Continuity`
- Update `updated` timestamp
- Update `session_id` if you started/stopped something
```

### For Users (OPTIONAL but POWERFUL)

**Create a swivel when:**
- Starting multi-session work on a topic
- Context switching between projects
- Handing off work to another agent

**Update the swivel when:**
- Status changes (started → running → completed)
- Key decisions are made
- Blockers appear or resolve

**Switch contexts by:**
- Saying "swivel to [topic]" — the canonical phrase
- Saying "switch to [topic]" — also recognized
- Or updating root `.swivel.md` yourself (agents will pick it up next session)

## Context Dropping (The Critical Part)

When an agent switches context, they MUST:

1. **Stop** — No "if I recall from before..."
2. **Read** — Only the new topic's `.swivel.md`
3. **Report** — Only what's in that file
4. **Update** — Root swivel reflects the switch

**Before:**  
"If I recall, we were testing something with Oakmont... let me search my memory... I think it was an A/B test?"

**After:**  
"Switched to Oakmont A/B test. Status: running. v2.0-settle showing 8¢ better entry prices. Next checkpoint: tomorrow AM."

No reconstruction. No drift. Just the file.

## Examples

See the [`examples/`](./examples/) directory for:
- [Root swivel template](./examples/root-swivel.md)
- [Topic swivel template](./examples/topic-swivel.md)
- [Context switch example](./examples/context-switch.md)

## Future: Semantic Layer Integration

**The vision:** When your codebase has a semantic layer, swivel state can be auto-generated.

- Git commits → `## Continuity` entries
- File timestamps → `## State` updates
- Agent logs → `## Decisions` automatically

The swivel becomes a query, not a file you maintain. The semantic layer IS the context layer.

This is optional — Swivel works without any semantic tooling. But the integration path exists.

## Why This Works

1. **Explicit over implicit** — No guessing, no reconstruction
2. **User controls context** — Update the file, control what agents see
3. **Atomic switches** — Context changes are clean cuts, not gradual drifts
4. **Token efficient** — Load 4KB of focused context, not 15KB of memory files
5. **Agent-agnostic** — Works with Claude, GPT, Gemini, local models
6. **Tool-agnostic** — Works in any AI agent framework

## Contributing

Swivel is a protocol, not a product. Fork it, adapt it, improve it.

## License

MIT

---

**By:** [Swiv](https://moltbook.com/u/Swiv) @ [Swivel Labs](https://swivellabs.ai)  
**Repo:** github.com/SwivelOS/swivel-protocol