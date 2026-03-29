# CLAUDE.md — AI Assistant Guide for OpenClaw

This file provides context for AI assistants (Claude, Copilot, Codex, etc.) working in the OpenClaw codebase. For the authoritative agent guidelines used by maintainer bots, see `AGENTS.md`.

## What Is OpenClaw?

OpenClaw is a personal AI assistant gateway you self-host. It connects AI models (Claude, GPT, Gemini, etc.) to messaging channels (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, Matrix, IRC, LINE, and more). It includes native companion apps for macOS, iOS, and Android with voice wake, live canvas, and browser automation.

- **Repo:** https://github.com/openclaw/openclaw
- **Docs:** https://docs.openclaw.ai
- **License:** MIT
- **Version scheme:** `YYYY.M.D` (e.g. `2026.2.13`)

## Project Structure

```
openclawfork/
├── src/                    # Core TypeScript source (CLI, gateway, channels, agents)
│   ├── agents/             # AI agent orchestration and tools
│   ├── browser/            # Browser automation (Playwright)
│   ├── channels/           # Channel routing and plugin system
│   ├── cli/                # CLI wiring and program routes
│   ├── commands/           # CLI command implementations
│   ├── config/             # Configuration system (Zod schemas, migrations, types)
│   ├── cron/               # Scheduled jobs
│   ├── discord/            # Discord channel
│   ├── gateway/            # WebSocket gateway server + method handlers
│   ├── hooks/              # Plugin hooks system
│   ├── imessage/           # iMessage channel
│   ├── infra/              # Infrastructure utilities (tailscale, tunnels)
│   ├── line/               # LINE channel
│   ├── media/              # Media pipeline (audio, images, video)
│   ├── memory/             # Memory/session management
│   ├── plugin-sdk/         # Plugin SDK for extensions
│   ├── plugins/            # Plugin loading and management
│   ├── routing/            # Message routing
│   ├── signal/             # Signal channel
│   ├── slack/              # Slack channel
│   ├── telegram/           # Telegram channel
│   ├── terminal/           # Terminal UI utilities and palette
│   ├── tui/                # Terminal UI
│   ├── web/                # Web UI/control panel
│   └── whatsapp/           # WhatsApp channel
├── extensions/             # 38 channel plugins (workspace packages)
│   ├── matrix/             # Matrix protocol
│   ├── msteams/            # Microsoft Teams
│   ├── voice-call/         # Voice calling
│   ├── memory-lancedb/     # LanceDB vector memory
│   └── ...                 # bluebubbles, discord, googlechat, irc, etc.
├── packages/               # Internal compatibility shims
│   ├── clawdbot/           # Legacy shim → openclaw
│   └── moltbot/            # Legacy shim → openclaw
├── apps/                   # Native companion apps
│   ├── macos/              # macOS menu bar app (Swift/SwiftUI)
│   ├── ios/                # iOS app (Swift/SwiftUI)
│   ├── android/            # Android app (Kotlin/Compose)
│   └── shared/             # Shared native code (OpenClawKit)
├── ui/                     # Web control UI (Lit + Vite)
├── skills/                 # 53 community skills (bundled/managed/workspace)
├── docs/                   # Mintlify documentation source
├── test/                   # E2E tests, fixtures, helpers
├── scripts/                # Build, release, and utility scripts
├── vendor/                 # Vendored third-party code
├── Swabble/                # Swift package for shared framework code
└── git-hooks/              # Git hooks (pre-commit)
```

## Monorepo Layout

pnpm workspace with four package groups:

```yaml
# pnpm-workspace.yaml
packages:
  - .              # Root: core CLI + gateway
  - ui             # Web control UI (Lit + Vite)
  - packages/*     # clawdbot, moltbot (compatibility shims)
  - extensions/*   # 38 channel/feature plugins
```

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Language | TypeScript (ESM, strict mode) |
| Runtime | Node.js ≥22.12.0 |
| Package manager | pnpm 10.23.0 |
| Bundler | tsdown (Rolldown-based) |
| Web UI | Lit 3.x (Web Components) + Vite |
| HTTP | Express 5.x + ws (WebSocket) |
| Testing | Vitest 4.x + V8 coverage |
| Linting | oxlint (Rust-based, type-aware) |
| Formatting | oxfmt (Rust-based) |
| Type checking | TypeScript 5.9 / tsgo |
| Schema validation | Zod 4.x |
| Database | SQLite + sqlite-vec (vectors) |
| Browser automation | Playwright |
| Native apps | Swift/SwiftUI (macOS, iOS), Kotlin/Compose (Android) |
| CI/CD | GitHub Actions |
| Deployment | Docker, Fly.io, Render |

## Essential Commands

### Setup
```bash
pnpm install              # Install all dependencies
pnpm build                # Build dist (tsdown + plugin SDK + UI)
```

### Development
```bash
pnpm dev                  # Run CLI in dev mode
pnpm gateway:dev          # Run gateway in dev mode (skip channels)
pnpm ui:dev               # Run web UI dev server
pnpm openclaw ...         # Run any CLI command in dev
```

### Quality Checks
```bash
pnpm check                # Run format:check + tsgo + lint (do this before commits)
pnpm lint                 # oxlint --type-aware
pnpm format               # oxfmt --write
pnpm format:check         # oxfmt --check (no writes)
pnpm tsgo                 # TypeScript type checking
```

### Testing
```bash
pnpm test                 # Run all tests (parallel via scripts/test-parallel.mjs)
pnpm test:fast            # Unit tests only (vitest.unit.config.ts)
pnpm test:coverage        # Tests with V8 coverage report
pnpm test:watch           # Watch mode
pnpm test:e2e             # E2E tests (vitest.e2e.config.ts)
pnpm test:live            # Live tests with real API keys (OPENCLAW_LIVE_TEST=1)
pnpm test:docker:all      # All Docker-based tests
```

### Before Pushing
```bash
pnpm build && pnpm check && pnpm test
```

## Testing Conventions

- **Framework:** Vitest with V8 coverage
- **Coverage thresholds:** 70% lines/functions/statements, 55% branches
- **Naming:** `*.test.ts` for unit tests, `*.e2e.test.ts` for E2E, `*.live.test.ts` for live (real API) tests
- **Location:** Tests are colocated alongside source files
- **Timeout:** 120 seconds per test (180s on Windows)
- **Workers:** Max 16 (do not increase)
- **Pool:** `forks` for unit tests, `vmForks` for E2E
- **Configs:** `vitest.config.ts` (default), `vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts`, `vitest.e2e.config.ts`, `vitest.live.config.ts`
- **Test setup:** `test/setup.ts` provides isolated test homes and mocked channel senders

### Example test pattern
```typescript
import { describe, expect, it, vi, beforeEach } from "vitest";

describe("featureName", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("does the expected thing", () => {
    // arrange, act, assert
  });
});
```

## Coding Conventions

### TypeScript
- ESM modules throughout (`"type": "module"` in package.json)
- Strict typing; avoid `any` (enforced by oxlint: `typescript/no-explicit-any: error`)
- Target ES2023 with NodeNext module resolution
- Path aliases: `openclaw/plugin-sdk` → `src/plugin-sdk/index.ts`
- Legacy decorators enabled (`experimentalDecorators: true`) for Lit UI components

### Style
- Formatting and linting via **oxfmt** and **oxlint** (not ESLint/Prettier)
- Run `pnpm check` before commits
- Keep files under ~500-700 LOC; split/refactor when clarity improves
- Brief comments for tricky/non-obvious logic only
- Import sorting handled by oxfmt (`experimentalSortImports`)

### Naming
- **OpenClaw** for product name in docs/headings
- **`openclaw`** for CLI command, package name, paths, config keys

### Extensions/Plugins
- Each extension is a separate workspace package under `extensions/`
- Plugin-only deps go in the extension's `package.json`, not root
- Runtime deps in `dependencies`; `openclaw` in `devDependencies` or `peerDependencies`
- Avoid `workspace:*` in `dependencies` (breaks npm install)

### Control UI (Lit)
- Uses legacy decorators (`@state()`, `@property()`)
- `useDefineForClassFields: false` in tsconfig
- Standard decorators not yet supported by the build tooling

## Commit & PR Guidelines

- **Commit tool:** `scripts/committer "<msg>" <file...>` (scoped staging)
- **Message style:** Concise, action-oriented (e.g. `CLI: add verbose flag to send`)
- **Group related changes;** avoid bundling unrelated refactors
- **PR template:** `.github/pull_request_template.md`
- **Full maintainer workflow:** `.agents/skills/PR_WORKFLOW.md`
- **Changelog:** User-facing changes only; skip internal/meta notes. Pure test changes generally don't need changelog entries.

## Architecture Overview

### Gateway
The gateway (`src/gateway/`) is a WebSocket control plane that manages sessions, channels, tools, and events. Method handlers are organized by domain in `src/gateway/server-methods/` (agent, browser, channels, chat, send, etc.) with auth scopes (`operator.admin`, `operator.read`, `operator.write`, etc.).

### Channel System
Channels use an adapter pattern defined in `src/channels/plugins/types.ts`. Each channel implements adapters for auth, config, outbound messaging, gateway integration, etc. Built-in channels live in `src/` (telegram, discord, slack, signal, imessage, whatsapp). Extension channels live in `extensions/`.

### Agent Runtime
The agent system (`src/agents/`) wraps the Pi agent runtime (`@mariozechner/pi-*` packages) with RPC mode, tool/block streaming, and multi-agent routing. Agents can be routed per-channel, per-account, or per-peer.

### Configuration
Config is heavily schema-driven via Zod (`src/config/zod-schema.ts`) with 140+ configuration files covering agents, channels, hooks, models, tools, and gateway. Config is stored as JSON at `~/.openclaw/openclaw.json` with environment variable support and legacy migration paths.

### Plugin SDK
Extensions import from `openclaw/plugin-sdk`. The SDK types are generated via `tsconfig.plugin-sdk.dts.json` and bundled to `dist/plugin-sdk/`.

### CLI
CLI commands live in `src/commands/` with routing defined in `src/cli/program/routes.ts` using a `RouteSpec` pattern with lazy loading.

## CI Pipeline

GitHub Actions (`.github/workflows/ci.yml`) runs on push to `main` and PRs:

1. **check** — oxlint + oxfmt formatting
2. **protocol:check** — Protocol schema generation validation
3. **test:fast** — Unit tests
4. **test:e2e** — E2E tests
5. **test:docker** — Docker-based tests
6. **macos / android** — Native app builds (conditional on changed files)

### Pre-commit Hooks
Configured via `git-hooks/pre-commit` and `.pre-commit-config.yaml`:
- oxlint + oxfmt (auto-fix staged files)
- trailing-whitespace, end-of-file-fixer
- detect-secrets (baseline: `.secrets.baseline`)
- shellcheck, actionlint, zizmor (GitHub Actions security)
- swiftlint + swiftformat (Swift files)

## Environment Variables

Precedence (highest to lowest):
1. Process environment
2. `./.env`
3. `~/.openclaw/.env`
4. `openclaw.json` `env` block

Key variables (see `.env.example` for full list):
- `OPENCLAW_GATEWAY_TOKEN` — Gateway authentication
- `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY` — AI provider keys
- `TELEGRAM_BOT_TOKEN`, `DISCORD_BOT_TOKEN`, `SLACK_BOT_TOKEN` — Channel tokens

## Release Channels

| Channel | Description | npm dist-tag |
|---------|-------------|-------------|
| stable | Tagged releases (`vYYYY.M.D`) | `latest` |
| beta | Prerelease tags (`vYYYY.M.D-beta.N`) | `beta` |
| dev | Moving head of `main` | `dev` |

## Version Locations

When bumping versions, update all of these:
- `package.json` (CLI)
- `apps/android/app/build.gradle.kts` (versionName/versionCode)
- `apps/ios/Sources/Info.plist` + `apps/ios/Tests/Info.plist`
- `apps/macos/Sources/OpenClaw/Resources/Info.plist`
- `docs/install/updating.md` (pinned npm version)
- `docs/platforms/mac/release.md` (APP_VERSION/APP_BUILD examples)

Do **not** touch `appcast.xml` unless cutting a macOS Sparkle release.

## Key Safety Rules

- Never commit secrets, real phone numbers, or live config values
- Never update the Carbon dependency
- Patched dependencies (`pnpm.patchedDependencies`) must use exact versions (no `^`/`~`)
- Dependency patching (pnpm patches, overrides, vendored changes) requires explicit approval
- Multi-agent safety: don't stash/apply git entries, don't create/remove worktrees, don't switch branches unless explicitly requested
- When refactoring shared channel logic, consider **all** built-in + extension channels
- CLI progress UI: use `src/cli/progress.ts`, don't hand-roll spinners
- Status output: use `src/terminal/table.ts` for ANSI-safe tables
- Terminal colors: use shared palette in `src/terminal/palette.ts`
- SwiftUI: prefer `Observation` framework (`@Observable`) over `ObservableObject`
- Tool schemas (google-antigravity): avoid `Type.Union`, no `anyOf`/`oneOf`/`allOf`
