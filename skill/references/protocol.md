# Swivel Protocol — Full Specification

This is the complete protocol specification. For quick reference, see SKILL.md.

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

## Context Dropping

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

## Why This Works

1. **Explicit over implicit** — No guessing, no reconstruction
2. **User controls context** — Update the file, control what agents see
3. **Atomic switches** — Context changes are clean cuts, not gradual drifts
4. **Token efficient** — Load 4KB of focused context, not 15KB of memory files
5. **Agent-agnostic** — Works with Claude, GPT, Gemini, local models
6. **Tool-agnostic** — Works in any AI agent framework