# OpenCode CLI Configuration

Personal configuration for the [OpenCode CLI](https://opencode.ai) with custom agents for planning, implementation, debugging, code review, research, and task orchestration.

This configuration is designed for **OpenCode Go only** — all models use the `opencode-go` provider.

Install it as the global OpenCode configuration at `~/.config/opencode`.

## Features

- Role-specific agents (see [Agents and Models](#agents-and-models))
- `orchestrator` as the default agent, delegating to specialized subagents
- Restricted agent permissions for safer automation
- Automatic loading of project instruction files (`AGENTS.md`, `CLAUDE.md`, `copilot-instructions.md`)

## Structure

```text
.
├── agents/
│   ├── builder.md
│   ├── fixer.md
│   ├── orchestrator.md
│   ├── planner.md
│   ├── researcher.md
│   └── reviewer.md
├── opencode.json
└── LICENSE
```

## Configuration

Defined in [`opencode.json`](./opencode.json):

- Provider: `opencode-go` (the only enabled provider)
- Default agent: `orchestrator`
- Maximum subagent depth: `1`

## Agents and Models

| Agent | Purpose | Mode | Model | Variant |
|---|---|---|---|---|
| `orchestrator` | Coordinates work and delegates tasks | `primary` | `opencode-go/deepseek-v4-flash` | — |
| `planner` | Read-only architectural analysis | `subagent` | `opencode-go/gpt-5.6-luna` | `hidden` |
| `builder` | Implements multi-file changes | `subagent` | `opencode-go/deepseek-v4-flash` | — |
| `fixer` | Handles small, localized fixes | `subagent` | `opencode-go/deepseek-v4-flash` | `low` |
| `reviewer` | Reviews code changes | `subagent` | `opencode-go/gpt-5.6-luna` | — |
| `researcher` | Answers technical questions and researches documentation | `primary` | `opencode-go/gpt-5.6-luna` | `low` |
| `docs` | Writes and maintains documentation grounded in the actual codebase | `subagent` | `opencode-go/gpt-5.6-luna` | `hidden` |

## Workflow

The `orchestrator`:

1. Reads project instructions and repository context
2. Delegates work to exactly one specialized subagent
3. Reviews the result
4. Runs applicable verification commands

## Installation

Clone this repository into OpenCode's configuration directory:

```bash
git clone https://github.com/joaortcoelho/opencode-config.git ~/.config/opencode
```

If the directory already exists, back it up first:

```bash
mv ~/.config/opencode ~/.config/opencode.backup
```

## License

See [`LICENSE`](./LICENSE).
