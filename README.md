# lazycc-codex-plugin

plugin for [lazy-claudecode](https://github.com/effortprogrammer/lazy-claudecode) — delegate tasks, run code reviews, and manage background jobs via OpenAI Codex CLI from within Claude Code.

> Derived from [codex-plugin-cc](https://github.com/openai/codex-plugin-cc) (MIT License, see [NOTICE](./NOTICE)).

## Features

| Command | Description |
|---------|-------------|
| `/codex:setup` | Check Codex CLI readiness, optionally install, toggle review gate |
| `/codex:review` | Run native Codex code review (foreground or background) |
| `/codex:adversarial-review` | Challenge review — questions design choices and tradeoffs |
| `/codex:rescue` | Delegate a task to Codex subagent (background or foreground) |
| `/codex:status` | Show tracked job status |
| `/codex:result` | Show stored final output of a completed job |
| `/codex:cancel` | Cancel an active job |
| `/codex:transfer` | Transfer Claude Code session into a resumable Codex thread |

### Hooks

- **SessionStart** — sets up session identity for job tracking
- **SessionEnd** — tears down broker, cleans up running jobs
- **Stop** — optional review gate: Codex reviews Claude's work before stopping (opt-in via `/codex:setup --enable-review-gate`)

## Prerequisites

- [OpenAI Codex CLI](https://github.com/openai/codex) installed globally: `npm install -g @openai/codex`
- Authenticated: `codex login`
- Node.js >= 18

## Installation

### Via lazy-claudecode (recommended)

When you install lazy-claudecode, this plugin is installed automatically:

```bash
npx lazy-claudecode install
```

### Standalone (Claude Code plugin system)

```bash
# From Claude Code, run:
/plugin install /path/to/lazycc-codex-plugin
```

Or clone and install:

```bash
git clone https://github.com/effortprogrammer/lazycc-codex-plugin.git
cd lazycc-codex-plugin
# Then in Claude Code:
/plugin install /path/to/lazycc-codex-plugin
```

## Architecture

```
lazycc-codex-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace listing
├── plugins/codex/
│   ├── .claude-plugin/
│   │   └── plugin.json           # Plugin identity
│   ├── hooks/
│   │   └── hooks.json            # Hook definitions
│   ├── commands/                  # Slash command definitions
│   │   ├── review.md
│   │   ├── adversarial-review.md
│   │   ├── rescue.md
│   │   ├── setup.md
│   │   ├── status.md
│   │   ├── result.md
│   │   ├── cancel.md
│   │   └── transfer.md
│   ├── agents/
│   │   └── codex-rescue.md       # Rescue subagent
│   ├── scripts/
│   │   ├── codex-companion.mjs   # Main CLI entry point
│   │   ├── app-server-broker.mjs # Shared app-server multiplexer
│   │   ├── session-lifecycle-hook.mjs
│   │   ├── stop-review-gate-hook.mjs
│   │   └── lib/                  # Core modules
│   ├── prompts/                  # Prompt templates
│   ├── schemas/                  # JSON schemas for structured output
│   └── skills/                   # Bundled skills
├── package.json
├── LICENSE
└── NOTICE
```

## How It Works

1. **Codex App-Server**: Communicates with `codex app-server` via JSON-RPC over stdin/stdout (or shared Unix socket broker for multiplexing)
2. **Job Tracking**: File-based persistence under `$CLAUDE_PLUGIN_DATA/state/` — tracks running, completed, failed jobs per workspace
3. **Session Scoping**: Jobs are tagged with `CODEX_COMPANION_SESSION_ID` from Claude Code's hook environment
4. **Broker Architecture**: A shared background process multiplexes app-server connections, allowing concurrent slash commands

## License

MIT — see [LICENSE](./LICENSE).

Derived from [codex-plugin-cc](https://github.com/openai/codex-plugin-cc) © 2025 OpenAI (MIT) — see [NOTICE](./NOTICE).
