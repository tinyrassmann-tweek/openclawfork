# Self-Improve Skill

Blake's self-improving loop. Run this daily (cron-triggered or manually invoked by Tiny).

## Purpose

Review the last 24 hours of session transcripts across all agents, identify what worked, what didn't, and propose targeted improvements to system prompts, HEARTBEAT.md, agent routing rules, or SOUL.md files. Output a concrete proposal — don't just describe the problem, recommend the fix.

## Trigger

Cron-scheduled: daily at 11PM (after the day's work is done, before morning brief generation).

Or manually: Tiny sends "run self-improve" to Blake via Telegram.

## Execution Steps

### 1. Gather Data

Review the following for the past 24 hours:

- `memory/YYYY-MM-DD.md` (today's log) — what tasks ran, what completed, what failed
- Sub-agent session logs if accessible:
  - `~/.openclaw/agents/coding-agent/memory/YYYY-MM-DD.md`
  - `~/.openclaw/agents/genesis-agent/memory/YYYY-MM-DD.md`
  - `~/.openclaw/agents/content-agent/memory/YYYY-MM-DD.md`
  - `~/.openclaw/agents/research-agent/memory/YYYY-MM-DD.md`

### 2. Identify Patterns

Look for:

- **Repeated clarification requests** — agent kept asking for info it should already know → update SOUL.md or USER.md
- **Missed tasks** — something was scheduled but not executed → check HEARTBEAT.md or cron config
- **Wrong agent routing** — task went to wrong sub-agent → update delegation rules in SOUL.md
- **Poor output quality** — deliverable needed heavy rework → update relevant agent's SOUL.md constraints
- **Token burn without value** — long agent runs that produced little → tighten prompts
- **Sub-agent communication gaps** — Blake didn't get a report back on time → check sessions_send routing

### 3. Generate Improvement Proposals

Format:

```
## Self-Improve Report — [YYYY-MM-DD]

### What worked well
- [2-3 items]

### Issues found
1. [Issue description]
   - Agent: [which agent]
   - Root cause: [what went wrong]
   - Proposed fix: [exact change to make — file, line, or config key]

### Files to update
- [ ] [filename] — [what to change]
- [ ] [filename] — [what to change]

### Priority
HIGH / MEDIUM / LOW
```

### 4. Apply or Escalate

- **LOW/MEDIUM**: Apply changes directly to SOUL.md, HEARTBEAT.md, or AGENTS.md files. Log what changed.
- **HIGH** (affects routing, config, or agent architecture): Send proposal to Tiny via Telegram for approval before applying.

### 5. Log the Run

Append to `memory/self-improve-log.md`:

```
## [YYYY-MM-DD HH:MM]
- Issues found: [count]
- Applied: [count]
- Escalated to Tiny: [count]
- Files changed: [list]
```

## Notes

- Don't change SOUL.md fundamentals without Tiny's approval
- Minor HEARTBEAT.md tuning (timing, checks) can be applied autonomously
- Config changes (openclaw.json) always require Tiny approval
- If nothing needs improving, log that too — "No issues found" is a valid output
