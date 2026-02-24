# Swivel Protocol v0.3

**State register, not planning doc.**

The Swivel Protocol is a lightweight, auto-updating state register for AI agents. It answers one question: "What's happening right now?"

## Quick Start

```bash
# Add to your agent workspace
curl -L https://raw.githubusercontent.com/SwivelOS/swivel-protocol/main/swivel-update > swivel-update
curl -L https://raw.githubusercontent.com/SwivelOS/swivel-protocol/main/swivel-pulse > swivel-pulse
chmod +x swivel-update swivel-pulse

# Create root swivel
cat > .swivel.md << 'EOF'
---
swivel: "0.3"
agent: myagent
updated: 2026-02-24T12:00:00-07:00
pulse: 2026-02-24T12:00:00-07:00
---

# State Register

## Active
(none)

## Watch
(none)

## Alerts
(none)

## Next
(none)
EOF

# Set up pulse cron (every 5 min)
echo "*/5 * * * * /path/to/swivel-pulse" | crontab -
```

## Architecture

### Root Swivel (`.swivel.md`)

20 lines max. Heartbeat reads it in <1 second.

```yaml
---
swivel: "0.3"
agent: swiv
updated: 2026-02-24T12:00:00-07:00
pulse: 2026-02-24T12:00:00-07:00
---

# State Register

## Active
- vanguard-build | SHIPPED | fleet/vanguard.swivel.md
- weather-trading | ACTIVE | trading/weather.swivel.md

## Watch
- vps-signal | OK | last: 2026-02-24T11:00:00-07:00

## Alerts
(none)

## Next
- Review Vanguard first synthesis
```

### Topic Swivels (`*.swivel.md`)

Live next to their work. Full context + staleness tracking.

```yaml
---
swivel: "0.3"
topic: weather-trading
status: ACTIVE
owner: alpha
touched: 2026-02-24T10:00:00-07:00
stale_after_hours: 12
---

# Weather Trading

## State
SDK built. Edge identified (60%+ on NYC).

## Files
- tools/weather_edge.py
- tools/kalshi_client.py

## Next
- Paper trade 5 markets
```

## Scripts

### `swivel-update`

Surgical state updates. Zero tokens.

```bash
swivel-update active vanguard-build SHIPPED
swivel-update watch vps-signal DOWN
swivel-update alert-add signal-gap "Signal missed 3 heartbeats"
swivel-update pulse
```

### `swivel-pulse`

Auto-checks that update root swivel:
- Penpal new letters
- Topic staleness (past `stale_after_hours`)
- VPS reachability
- Pulse timestamp

## Design Principles

1. **State register, not planning doc.** Answers "what's happening now?"
2. **20 lines max for root.** Heartbeat reads in <1 second.
3. **Auto-update hooks.** State changes write automatically.
4. **Fleet-wide standard.** Same format everywhere.
5. **Tiered detail.** Root = pointers. Topics = full context.

## Migration from v0.2

See [MIGRATION.md](MIGRATION.md).

## License

MIT — See [LICENSE](LICENSE)

---

*Swivel Protocol — Context without bloat.*
