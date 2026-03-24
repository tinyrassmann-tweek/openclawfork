# DynastyHoldings — openclaw.json Configuration Guide

This document contains the exact `openclaw.json` snippets to implement the DynastyHoldings
Master Build Plan. Apply these on the Hostinger VPS to your live config file.

**Config location on VPS:** `~/.openclaw/openclaw.json` (or wherever your config is mounted)

---

## STEP 1: Pull v2026.3.2 Update (ClawJacked security patches)

When the upstream releases v2026.3.2, run on VPS:

```bash
cd /path/to/openclaw
git fetch origin
git pull origin main
pnpm install
pnpm build
# Then restart the OpenClaw service
pm2 restart openclaw   # or however you manage the process
```

Current codebase is v2026.2.13. The fork is already at the latest available commit.
Watch the upstream repo for the next release tag.

---

## STEP 2: Remove Ngrok → Use Hostinger VPS URL Directly

**Remove or disable the voice-call extension's tunnel config.** If you're not using Twilio voice calls, disable the extension entirely:

```json
{
  "extensions": {
    "voice-call": {
      "enabled": false
    }
  }
}
```

If you ARE using voice calls, replace the Ngrok config with your static VPS URL:

```json
{
  "extensions": {
    "voice-call": {
      "enabled": true,
      "tunnel": {
        "mode": "static",
        "staticUrl": "https://YOUR-VPS-DOMAIN-OR-IP:PORT"
      }
    }
  }
}
```

---

## STEP 3: Lock Down Exec Auto-Approval (AUTOMERGE_PATHS → /logs/* only)

In OpenClaw, "AUTOMERGE_PATHS" maps to the exec tool's security mode. The following config
sets the exec tool to **allowlist mode** — agents must be explicitly approved for commands
that write outside of /logs/. The `ask: "on-miss"` triggers a Telegram approval prompt for
any command not on the safe list.

```json
{
  "tools": {
    "exec": {
      "security": "allowlist",
      "ask": "on-miss",
      "safeBins": ["cat", "ls", "grep", "head", "tail", "echo", "date", "pwd"],
      "timeoutSec": 120,
      "notifyOnExit": true
    }
  }
}
```

For path-level write restrictions, apply per-agent sandbox scope:

```json
{
  "agents": {
    "list": [
      {
        "id": "blake",
        "sandbox": {
          "mode": "non-main",
          "workspaceAccess": "rw",
          "scope": "agent"
        },
        "tools": {
          "exec": {
            "security": "allowlist",
            "ask": "on-miss"
          }
        }
      }
    ]
  }
}
```

**In practice:** Any agent command that would write outside the workspace or /logs/ will
ping you on Telegram for approval. This is the OpenClaw equivalent of AUTOMERGE_PATHS.

---

## STEP 4: Add LFM2-24B via Ollama

First, pull the model on the VPS:

```bash
ollama pull lfm2-24b
# Verify it's available:
ollama list
```

Then register it in `openclaw.json` under `models.providers`:

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "ollama": {
        "baseUrl": "http://localhost:11434",
        "api": "ollama",
        "models": [
          {
            "id": "lfm2-24b",
            "name": "LFM2-24B (Local)",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 32768,
            "maxTokens": 8192,
            "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 }
          }
        ]
      }
    }
  }
}
```

To assign LFM2-24B as the default model for Genesis Agent (HIPAA-safe local processing):

```json
{
  "agents": {
    "list": [
      {
        "id": "genesis-agent",
        "model": "ollama/lfm2-24b"
      }
    ]
  }
}
```

---

## STEP 5: Full Multi-Agent Configuration (Muddy OS Architecture)

This is the complete Blake + 4 sub-agents setup. Blake is the default agent (COO/Orchestrator).
The four sub-agents are spawnable by Blake only.

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "enabled": true,
        "everyMs": 1800000
      }
    },
    "list": [
      {
        "id": "blake",
        "default": true,
        "name": "Blake",
        "model": "anthropic/claude-sonnet-4-6",
        "skills": ["self-improve", "PR_WORKFLOW"],
        "subagents": {
          "allowAgents": ["coding-agent", "genesis-agent", "content-agent", "research-agent"]
        },
        "tools": {
          "exec": {
            "security": "allowlist",
            "ask": "on-miss"
          }
        }
      },
      {
        "id": "coding-agent",
        "name": "Forge",
        "model": "anthropic/claude-sonnet-4-6",
        "subagents": {
          "allowAgents": []
        },
        "tools": {
          "exec": {
            "security": "allowlist",
            "ask": "on-miss",
            "host": "gateway"
          }
        }
      },
      {
        "id": "genesis-agent",
        "name": "Genesis",
        "model": "ollama/lfm2-24b",
        "subagents": {
          "allowAgents": []
        },
        "tools": {
          "exec": {
            "security": "deny"
          }
        }
      },
      {
        "id": "content-agent",
        "name": "Scout",
        "model": "anthropic/claude-sonnet-4-6",
        "subagents": {
          "allowAgents": []
        }
      },
      {
        "id": "research-agent",
        "name": "Pulse",
        "model": "anthropic/claude-sonnet-4-6",
        "subagents": {
          "allowAgents": []
        }
      }
    ]
  }
}
```

---

## STEP 6: Cron Jobs (Morning Brief + Self-Improve Loop + Executive Sync)

Cron jobs are stored in the cron store file, NOT directly in `openclaw.json`.
They're managed via Blake's chat interface or by editing the cron store file.

**Cron store location:** `~/.openclaw/agents/blake/cron.json`

Create or edit that file with:

```json
{
  "version": 1,
  "jobs": [
    {
      "id": "morning-brief",
      "agentId": "blake",
      "name": "Morning Brief",
      "description": "Daily 8AM brief to Tiny via Telegram",
      "enabled": true,
      "createdAtMs": 1742760000000,
      "updatedAtMs": 1742760000000,
      "schedule": {
        "kind": "cron",
        "expr": "0 8 * * *",
        "tz": "America/Chicago"
      },
      "sessionTarget": "main",
      "wakeMode": "now",
      "payload": {
        "kind": "agentTurn",
        "message": "Read HEARTBEAT.md and deliver the morning brief to Tiny on Telegram. Include: yesterday's wins, today's top 3 priorities, any blockers needing a decision. Keep it concise — 10 lines max. Send via Telegram.",
        "deliver": true,
        "channel": "telegram",
        "bestEffortDeliver": false
      },
      "state": {}
    },
    {
      "id": "self-improve-loop",
      "agentId": "blake",
      "name": "Self-Improve Loop",
      "description": "Daily 11PM review of sessions and improvement proposals",
      "enabled": true,
      "createdAtMs": 1742760000000,
      "updatedAtMs": 1742760000000,
      "schedule": {
        "kind": "cron",
        "expr": "0 23 * * *",
        "tz": "America/Chicago"
      },
      "sessionTarget": "isolated",
      "wakeMode": "now",
      "payload": {
        "kind": "agentTurn",
        "message": "Run the self-improve skill. Review today's memory log and all sub-agent logs. Identify issues. Propose and apply LOW/MEDIUM improvements. Escalate HIGH changes to Tiny via Telegram for approval.",
        "deliver": true,
        "channel": "telegram",
        "bestEffortDeliver": true
      },
      "state": {}
    },
    {
      "id": "executive-sync",
      "agentId": "blake",
      "name": "Evening Executive Sync",
      "description": "Daily 6PM sub-agent status summary to Tiny",
      "enabled": true,
      "createdAtMs": 1742760000000,
      "updatedAtMs": 1742760000000,
      "schedule": {
        "kind": "cron",
        "expr": "0 18 * * *",
        "tz": "America/Chicago"
      },
      "sessionTarget": "isolated",
      "wakeMode": "now",
      "payload": {
        "kind": "agentTurn",
        "message": "Generate the evening executive sync. Check the memory logs of Forge, Genesis, Scout, and Pulse for today. Summarize: what each agent completed, any issues, what carries over to tomorrow. Send a concise summary to Tiny via Telegram.",
        "deliver": true,
        "channel": "telegram",
        "bestEffortDeliver": true
      },
      "state": {}
    }
  ]
}
```

**Alternative:** Add cron jobs via Blake's Telegram chat interface:
```
/cron add daily at 8:00 AM: run morning brief and send to Telegram
```

---

## STEP 7: Deploy Agent Workspace Files

After pushing this repo, deploy the workspace files to each agent's directory on the VPS:

```bash
# Create agent workspace directories
mkdir -p ~/.openclaw/agents/blake
mkdir -p ~/.openclaw/agents/coding-agent
mkdir -p ~/.openclaw/agents/genesis-agent
mkdir -p ~/.openclaw/agents/content-agent
mkdir -p ~/.openclaw/agents/research-agent

# Deploy IDENTITY.md and SOUL.md for each agent
# (run from the openclawfork repo directory)
cp .dynasty/agents/blake/IDENTITY.template.md ~/.openclaw/agents/blake/IDENTITY.md
cp .dynasty/agents/blake/SOUL.md ~/.openclaw/agents/blake/SOUL.md
cp .dynasty/agents/blake/HEARTBEAT.md ~/.openclaw/agents/blake/HEARTBEAT.md

cp .dynasty/agents/coding-agent/IDENTITY.template.md ~/.openclaw/agents/coding-agent/IDENTITY.md
cp .dynasty/agents/coding-agent/SOUL.md ~/.openclaw/agents/coding-agent/SOUL.md

cp .dynasty/agents/genesis-agent/IDENTITY.template.md ~/.openclaw/agents/genesis-agent/IDENTITY.md
cp .dynasty/agents/genesis-agent/SOUL.md ~/.openclaw/agents/genesis-agent/SOUL.md

cp .dynasty/agents/content-agent/IDENTITY.template.md ~/.openclaw/agents/content-agent/IDENTITY.md
cp .dynasty/agents/content-agent/SOUL.md ~/.openclaw/agents/content-agent/SOUL.md

cp .dynasty/agents/research-agent/IDENTITY.template.md ~/.openclaw/agents/research-agent/IDENTITY.md
cp .dynasty/agents/research-agent/SOUL.md ~/.openclaw/agents/research-agent/SOUL.md
```

---

## STEP 8: Memory Fix Prompt (Agent System Prompt Hook)

Apply this as a system prompt hook to improve Blake's cross-session memory recall.
Add to `openclaw.json` under the `hooks` section:

```json
{
  "hooks": {
    "systemPrompt": {
      "append": "At the start of every session: (1) Read SOUL.md to re-establish who you are. (2) Read USER.md to re-establish who you serve. (3) Read memory/YYYY-MM-DD.md for today and yesterday. (4) Check HEARTBEAT.md for pending tasks. Do not ask for permission to read these files — just do it. Your memory lives in files, not in this conversation."
    }
  }
}
```

---

## STEP 9: Telegram Channel Binding

Bind Blake to your Telegram bot. In `openclaw.json`:

```json
{
  "telegram": {
    "accounts": [
      {
        "id": "blake-telegram",
        "token": "${TELEGRAM_BOT_TOKEN}",
        "enabled": true
      }
    ]
  },
  "routing": {
    "bindings": [
      {
        "agentId": "blake",
        "match": {
          "channel": "telegram",
          "accountId": "blake-telegram"
        }
      }
    ]
  }
}
```

---

## Full Verification Checklist

After applying all config changes and restarting OpenClaw:

- [ ] **Dashboard**: Open `http://YOUR-VPS-IP:18789` — Mission Control UI loads
- [ ] **Telegram**: Send "hello" to Blake's bot — gets a response with name "Blake" and 🦅 emoji
- [ ] **Ollama**: `ollama list` on VPS shows `lfm2-24b`
- [ ] **Sub-agent test**: Tell Blake "spawn Forge and have it check disk space" → Forge executes → Blake reports back
- [ ] **Morning brief**: Wait for 8AM or trigger manually: "run morning brief now"
- [ ] **Exec approval**: Ask Blake to write a file outside /logs/ → Telegram approval prompt appears
- [ ] **Genesis local**: Ask Genesis Agent to process a document → verify it uses Ollama, not Anthropic API
- [ ] **Self-improve**: Tell Blake "run self-improve" → gets a report back via Telegram
- [ ] **Identity**: Ask each agent "who are you?" → each gives their correct name and role

---

## Environment Variables Required on VPS

```bash
# In your .env or system environment:
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
ANTHROPIC_API_KEY=your_anthropic_api_key
# Ollama runs locally — no API key needed
```

---

## Week-by-Week Remaining Work (Outside This Repo)

| Week | Items | Location |
|------|-------|----------|
| W3 | Gemini CLI install | Terminal on VPS |
| W3 | Google Flows setup | Google Workspace |
| W3 | NotebookLM chatbot | Google NotebookLM |
| W3 | Gmail AI features | Google Workspace |
| W4 | Goose backtesting | Separate install ($99) |
| W4 | Pine Script import | TradingView |
| W4 | Topstep MGC deployment | Topstep platform |
| W5 | MyClaw.ai client workflow | MyClaw.ai ($40/mo) |
| W5 | WhatsApp client number | WhatsApp Business |
