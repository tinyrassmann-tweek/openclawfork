# DynastyHoldings / OpenClaw — Session Memory Note

**Written:** 2026-06-23, by Claude (remote sandbox session on `openclawfork` repo)
**Purpose:** Persistent record for the next session/instance to read before continuing work.

---

## IMPORTANT — Read This First

This note was written from a **remote cloud sandbox session** operating on the
`tinyrassmann-tweek/openclawfork` git repo. It has **no access to the user's local Mac
filesystem** (`/Users/homemac/...`). It cannot read or write `/homemac/Claude.html` or
any other local Mac file directly.

If continuity across the Mac-local Claude Code session and this repo-based session is
needed, the user must manually carry this file's contents between them (paste it into
the local memory file, or have the next local session `git pull` this repo and read
`.dynasty/SESSION-NOTES.md` directly).

---

## What's ACTUALLY Done (verified, in the repo, real)

**Branch:** `claude/video-tutorial-implementation-GhRGq`
**PR:** [#2](https://github.com/tinyrassmann-tweek/openclawfork/pull/2) — open, clean, mergeable, no CI configured, no review comments as of last check.

13 files committed and pushed:
- `.dynasty/CONFIG-GUIDE.md` — 9-step deployment guide covering: OpenClaw version update, Ngrok→VPS URL swap, exec security lockdown (AUTOMERGE_PATHS equivalent), Ollama/LFM2-24B registration, full multi-agent JSON config, 3 cron jobs (8AM brief / 6PM sync / 11PM self-improve), memory system-prompt hook, Telegram binding, VPS deploy commands.
- `.dynasty/agents/{blake,coding-agent,genesis-agent,content-agent,research-agent}/SOUL.md` — operating principles per agent
- `.dynasty/agents/{blake,coding-agent,genesis-agent,content-agent,research-agent}/IDENTITY.template.md` — name/persona per agent (named `.template.md` because `IDENTITY.md` is globally gitignored in this repo)
- `.dynasty/agents/blake/HEARTBEAT.md` — morning brief + sub-agent health check logic
- `.agents/skills/self-improve.md` — daily self-review skill definition

**This is all documentation and config templates living IN THE REPO. None of it is deployed
to the actual Hostinger VPS yet.** Nothing has touched the live `openclaw.json`, no cron
jobs actually exist, no Ollama model has been pulled, no Telegram bot is bound.

## What's NOT Done

- Hostinger VPS: no SSH session was ever established in this conversation. No IP/credentials provided.
- Ollama `lfm2-24b`: not pulled anywhere (was only documented as a step).
- Cron jobs: only exist as JSON snippets in CONFIG-GUIDE.md — not created on any real cron store.
- Telegram bot binding: not live.
- `agent-browser` on the Mac: npm install completed, Chrome confirmed present (v146.0.7680.165).
  MCP wiring into `~/.claude/settings.json` was given as instructions but **last confirmed `/mcp`
  check on the Mac showed only Figma/Gmail/Google Calendar/Slack connected — agent-browser was
  NOT in that list.** Either the settings.json edit didn't happen, or Claude Code wasn't restarted
  after editing it, or it failed silently. Needs re-verification.

## Known State Confusion This Session

- User reports OpenClaw has been **deleted from the Mac hard drive for months** — any
  Mac-side OpenClaw work needs a reinstall first. All real progress this session happened
  in this remote repo sandbox, NOT on the user's Mac.
- The remote sandbox session (this one) and the user's local Mac Claude Code terminal
  session are **two completely separate environments.** Files written in one are invisible
  to the other. This caused real confusion earlier in the conversation (agent-browser install
  commands run on the wrong machine, confusion about which terminal does what).
- Mid-session, several MCP servers disconnected/reconnected (Base44, Context7, Intuit_TurboTax,
  Shopify dropped; github reconnected). This is normal MCP churn, not an error to fix.
- "Auto Mode" was toggled on by the user — future turns in this kind of session should act
  rather than pause for confirmation, except on genuinely ambiguous or destructive decisions.

## Stack As Currently Understood

| Component | Status | Where |
|---|---|---|
| openclawfork repo | PR #2 open, unmerged | GitHub, this sandbox |
| Hostinger VPS | Not yet touched this session | Needs SSH access |
| Blake + 4 sub-agents (Forge/Genesis/Scout/Pulse) | Designed, documented, NOT live | `.dynasty/` in repo |
| agent-browser (Mac) | Installed, MCP wiring unconfirmed | User's Mac |
| OpenClaw (Mac) | Reportedly deleted/missing | User's Mac |
| Telegram, Ollama/LFM2-24B | Documented only, not deployed | N/A |

## Recommended Next Steps

1. Decide: is all further OpenClaw/Dynasty work happening VPS-side via this repo, or does
   the Mac also need a fresh OpenClaw install? Pick one before continuing.
2. Get actual Hostinger SSH access into a session to deploy PR #2's CONFIG-GUIDE.md steps for real.
3. Re-verify `agent-browser` MCP connection on the Mac with a fresh `/mcp` check.
4. Decide whether to merge PR #2 now (it's just docs/templates, low risk) or hold it.
