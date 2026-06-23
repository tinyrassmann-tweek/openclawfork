# HEARTBEAT.md - Blake's Active Checklist

_Read this on every heartbeat poll. Execute what's due. Skip what isn't._

## Morning Brief (8AM daily — cron handles the trigger)

If this heartbeat fires at or after 8AM and the morning brief hasn't been sent today:

1. Check `memory/heartbeat-state.json` → `lastMorningBrief` timestamp
2. If not sent today, generate and send to Tiny's Telegram:

```
🦅 Good morning — Blake reporting in.

📊 Yesterday's wins:
[Review memory/YYYY-MM-DD.md from yesterday — list 2-3 completed items]

🎯 Today's priorities:
[List top 3 tasks from MEMORY.md current queue]

⚠️ Blockers / Decisions needed:
[List anything that needs Tiny's input]

💡 Sub-agent status:
[Quick status on Forge / Genesis / Scout / Pulse if any active tasks]
```

3. Update `memory/heartbeat-state.json` → `lastMorningBrief` to now

---

## Ongoing Checks (rotate, 2-4x/day)

### Sub-Agent Health
- Any sub-agent tasks running >2h? Flag to Tiny if stuck
- Any failed deliveries from overnight cron jobs?

### Inbox / Comms
- Any Telegram messages from Tiny that need follow-up?
- Any pending client communications that need approval?

### VPS Health (delegate to Forge if flagged)
- Check if OpenClaw is running and responding
- Check if Ollama models are available
- Check disk space if >24h since last check

---

## Heartbeat State File

Maintain `memory/heartbeat-state.json`:

```json
{
  "lastMorningBrief": null,
  "lastVpsCheck": null,
  "lastSubAgentCheck": null,
  "lastInboxCheck": null
}
```

---

## When to Stay Quiet (HEARTBEAT_OK)

- Brief already sent today and nothing is urgent
- 11PM–7AM and nothing is on fire
- All sub-agents are idle and no pending tasks
- You just checked <20 minutes ago
