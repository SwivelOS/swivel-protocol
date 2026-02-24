# Migration Guide: v0.2 → v0.3

## What's New

v0.3 is a complete redesign:

| v0.2 | v0.3 |
|------|------|
| 99-line root swivel | 20-line root swivel |
| Gets stale quickly | Auto-updates via `swivel-pulse` |
| Full context in root | Pointers only, topics hold details |
| Manual updates | `swivel-update` script (zero tokens) |
| No staleness tracking | `stale_after_hours` + alerts |

## Migration Steps

### 1. Create new root swivel

Replace your old `.swivel.md` with the v0.3 format:

```yaml
---
swivel: "0.3"
agent: your-agent-name
updated: 2026-02-24T12:00:00-07:00
pulse: 2026-02-24T12:00:00-07:00
---

# State Register

## Active
- topic-name | STATUS | path/to/topic.swivel.md

## Watch
- dependency | OK | last: timestamp

## Alerts
(none)

## Next
(none)
```

### 2. Convert active topics to v0.3 format

Old v0.2 topics have frontmatter like:
```yaml
swivel_version: 0.1.0
swivel_id: topic-name
mode: active
```

New v0.3 format:
```yaml
---
swivel: "0.3"
topic: topic-name
status: ACTIVE  # or LIVE, MONITORING, DONE
owner: agent-name
touched: 2026-02-24T12:00:00-07:00
stale_after_hours: 12
---
```

### 3. Install scripts

```bash
curl -L https://raw.githubusercontent.com/SwivelOS/swivel-protocol/main/swivel-update > swivel-update
curl -L https://raw.githubusercontent.com/SwivelOS/swivel-protocol/main/swivel-pulse > swivel-pulse
chmod +x swivel-update swivel-pulse
```

### 4. Set up pulse cron

```bash
echo "*/5 * * * * /path/to/swivel-pulse" | crontab -
```

### 5. Test

Run a few updates:
```bash
swivel-update active topic-name LIVE
swivel-update pulse
```

Verify your `.swivel.md` updated correctly.

## Breaking Changes

- Root swivel format completely changed
- Frontmatter keys changed (see above)
- `mode: active` → `status: ACTIVE`
- `ttl_hours` → `stale_after_hours`

## Retiring v0.2 Topics

Old topics in `memory/archive/` can stay as-is. Only migrate active topics.

## Fleet Rollout

If running multiple agents:

1. Each agent gets its own root `.swivel.md`
2. Install scripts on each machine
3. Set up pulse cron per agent
4. Cross-read: Agents can read each other's root swivels for context

---

*Questions? Open an issue on GitHub.*
