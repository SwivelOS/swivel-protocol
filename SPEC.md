---
swivel_version: 0.2.0
swivel_id: swivel-spec-v2
swivel_type: spec
---

# Swivel Specification v0.2

**Session Context Protocol with Reality Validation**

---

## What's New in v0.2

| Feature | v0.1 | v0.2 |
|---------|------|------|
| **TTL (time-to-live)** | ❌ Manual staleness | ✅ Auto-expire after N hours |
| **Healthcheck integration** | ❌ Declarations only | ✅ Link to process/commands |
| **last_verified** | ❌ Trust timestamps | ✅ Verify reality |
| **Drift taxonomy** | ❌ Generic "stale" | ✅ Typed drift detection |
| **Status enum** | active/paused/completed | ✅ + archived, error, blocked |
| **CLI tooling** | ❌ Read-only | ✅ Query, verify, archive |

**Breaking change:** v0.2 swivels are backward-compatible but v0.1 agents won't use new fields.

---

## The Problem

Every AI agent session starts the same way:

- **User:** "Hey, can you check on that thing?"
- **Agent:** "What thing?"  
- **User:** "The thing from yesterday. The trading bot."
- **Agent:** *searches memory, reconstructs state, guesses wrong*

**Swivel v0.1 solved the reconstruction problem.**  
**Swivel v0.2 solves the *trust* problem.**

Because v0.1 had a flaw: swivels lie. My swivel said Alpha was "running" — he wasn't. Stale context is worse than no context.

---

## Core Concepts

### Swivel File

A `.swivel.md` file is a **verified context declaration**. It tells an agent:
- What state to load
- What files to read
- What the current status is
- **When it was last verified against reality**
- **What kind of drift (if any) exists**

### Reality Verification

Every swivel can define a `healthcheck` — a command or set of assertions that verify declarations match reality:

```yaml
healthcheck:
  type: command  # command | file | http | custom
  command: "pgrep -f 'workspace-alpha'"
  expected: ">0"
  interval_minutes: 5
```

### Drift Taxonomy

When swivel ≠ reality, classify the drift:

| Drift Type | Meaning | Auto-Resolve? |
|------------|---------|---------------|
| `none` | Swivel matches reality | N/A |
| `ttl_expired` | Swivel older than TTL | ❌ Requires re-verify |
| `process_mismatch` | Declared process not running | ❌ Check if crashed/paused |
| `file_missing` | Referenced file deleted/moved | ❌ Update paths |
| `topic_orphan` | Active topic with no recent work | ⚠️ Warn, auto-archive after N days |
| `handoff_failed` | Recipient never acknowledged | ❌ Escalate to parent |
| `blocker_stale` | Blocker >24h without update | ❌ Escalate to owner |

---

## File Format (v0.2)

```markdown
---
swivel_version: 0.2.0
swivel_id: oakmont-ab-test
swivel_type: topic|experiment|draft|spec
session_id: abc123-...-xyz
session_target: main|isolated
created: 2026-02-17T22:00:00Z
updated: 2026-02-17T22:30:00Z
updated_by: swiv
mode: active|paused|completed|archived|error|blocked
parent: trading

# NEW in v0.2: TTL and verification
ttl_hours: 24                    # Auto-stale after 24h
last_verified: 2026-02-19T11:00:00Z  # When reality was checked
verified_by: omega               # Who checked

# NEW in v0.2: Healthcheck integration
healthcheck:
  type: command
  command: "pgrep -f 'oakmont'"
  expected: ">0"
  interval_minutes: 60
  last_check: 2026-02-19T11:00:00Z
  check_result: pass|fail|error

# NEW in v0.2: Drift tracking
drift:
  status: none                   # none | ttl_expired | process_mismatch | ...
  detected_at: null
  severity: info|warn|error|critical
  details: null
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

---

## Protocol Rules (v0.2)

### For Agents (MANDATORY)

**1. Read `.swivel.md` first — AND verify TTL**
```
Every new session MUST:
1. Read ./.swivel.md
2. Check if ttl_hours has expired
3. If expired: RE-VERIFY before trusting
4. Report drift.status if not "none"
```

**2. Load ONLY what's listed**
```
The ## Load section is complete. Don't load files not listed.
Don't search memory. The swivel IS source of truth — IF verified.
```

**3. Report with verification status**
```
When asked "what are we working on?", report:
"Oakmont A/B test. Status: running (verified 11:00). 
 11 trades v2.0, 4 v1.0. Next: tomorrow AM."

If drift detected:
"Oakmont A/B test. ⚠️ DRIFT: process_mismatch. 
 Swivel says running, but no processes found. Needs verification."
```

**4. Drop context on switch**
```
When switching:
1. STOP referencing old topic
2. READ new topic's .swivel.md (check TTL!)
3. UPDATE root .swivel.md
4. CONFIRM with verification status
```

**5. Update BEFORE ending — AND verify health**
```
Before ending a session where you did work:
- Append to ## Continuity
- Update updated timestamp
- Run healthcheck if defined
- Update drift.status based on reality
- Update last_verified
```

### For Fleet Operators

**6. Verify, don't trust**
```
Every heartbeat:
1. Check ttl_hours against last_verified
2. Run healthcheck if defined
3. Update drift.status
4. Escalate if drift.severity ≥ error
```

**7. Drift handling by type**

| Drift | Action |
|-------|--------|
| `ttl_expired` | Re-verify before use. Don't trust stale. |
| `process_mismatch` | Check if intentionally paused. Update swivel. |
| `topic_orphan` | Warn owner. Auto-archive after 7 days. |
| `handoff_failed` | Escalate to parent. Don't proceed. |
| `blocker_stale` | Ping blocker owner. Update ETA or reassign. |

---

## Root Swivel (v0.2)

```markdown
---
swivel_version: 0.2.0
swivel_id: root
swivel_type: topic
session_id: 981278ef-7919-433c-ac7c-aae38b8f3a0c
created: 2026-02-17T22:00:00Z
updated: 2026-02-18T15:25:00Z
updated_by: swiv
mode: normal
ttl_hours: 24
last_verified: 2026-02-19T11:00:00Z
verified_by: omega
drift:
  status: none
---

## Load
- ./SOUL.md
- ./USER.md
- ./AGENTS.md

## Active Topics
- [oakmont-ab-test](./trading/oakmont-ab-test.swivel.md)
  status: paused
  verified: 2026-02-19T11:00:00Z
  drift: process_mismatch  # Swivel says running, no processes found
  
- [prism-packaging](./prism/prism-packaging.swivel.md)
  status: active
  verified: 2026-02-19T10:30:00Z
  drift: none

## Paused Topics
- [solana-trading](./trading/solana/solana-pause.swivel.md)
  status: paused
  drift: topic_orphan  # No activity for 14 days

## Archived Topics
- [old-experiment](./archive/old-experiment.swivel.md)
  status: archived
  completed: 2026-02-10
```

---

## Swivel CLI (v0.2)

Command-line tool for fleet operators:

```bash
# Check status of any swivel
swivel status --topic=oakmont-ab-test
# → Status: paused (drift: process_mismatch, verified 11:00)

# Verify against reality
swivel verify --topic=oakmont-ab-test
# → Running healthcheck: pgrep -f 'oakmont'
# → Result: fail (0 processes)
# → Drift detected: process_mismatch
# → Updated swivel drift.status

# List all drifts
swivel drift --since=24h
# → oakmont-ab-test: process_mismatch (11:00)
# → prism-packaging: none (10:30)
# → solana-trading: topic_orphan (7 days)

# Archive old topics
swivel archive --topic=solana-trading --reason="14 days inactive"

# Fleet-wide health check
swivel fleet-health
# → 4 agents checked
# → 1 drift detected (oakmont-ab-test: process_mismatch)
# → 1 stale (forge: ttl_expired)
```

---

## Migration Guide: v0.1 → v0.2

**Step 1: Update frontmatter**
```diff
  ---
  swivel_version: 0.1.0
+ swivel_version: 0.2.0
+ ttl_hours: 24
+ last_verified: 2026-02-19T11:00:00Z
+ drift:
+   status: none
  ---
```

**Step 2: Add healthchecks to critical swivels**
```yaml
healthcheck:
  type: command
  command: "pgrep -f 'process-name'"
  expected: ">0"
```

**Step 3: Update agents to check TTL**
```python
# Pseudocode
def load_swivel(path):
    swivel = parse_yaml_frontmatter(path)
    if swivel.ttl_hours:
        age = now() - swivel.last_verified
        if age > timedelta(hours=swivel.ttl_hours):
            print(f"⚠️ Swivel expired. Re-verifying...")
            verify_swivel(swivel)
    return swivel
```

---

## Why v0.2 Works Better

| v0.1 Problem | v0.2 Solution |
|--------------|---------------|
| Swivel says "running" but process crashed | `healthcheck` + `drift: process_mismatch` |
| 3-day-old swivel trusted as current | `ttl_hours` + `last_verified` forces re-check |
| Stale topics accumulate forever | `drift: topic_orphan` + auto-archive |
| No way to query fleet state | `swivel fleet-health` gives snapshot |
| Handoff failures go unnoticed | `drift: handoff_failed` + escalation |

---

## Success Metrics

- **Zero** trusted-but-false swivels
- **<5min** to verify entire fleet health
- **100%** of handoffs logged and confirmed
- **<1%** drift rate (ideally 0%)

---

## Contributing

**Protocol Owner:** Omega (Ω) — Fleet Operator  
**Original Author:** Swiv (Swivel Labs)  
**License:** MIT  
**Repo:** github.com/SwivelOS/swivel-protocol

---

## See Also

- [README.md](./README.md) — Overview and quick start
- [Swivel v0.1 Spec](./SWIVEL-SPEC-v0.1.md) — Original protocol (if archived)

*"Trust, but verify. The swivel knows — when it's verified."*
