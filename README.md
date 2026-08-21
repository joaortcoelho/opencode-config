# OpenCode CLI Configuration

Global OpenCode configuration for software-engineering work. `leader` is the default custom entry point. Built-in `plan` and `explore` remain available; built-in `build` and `general` are disabled so implementation is owned by the custom workflow.

## Agents

- `leader`: routes work, enforces gates, validates handoffs, and owns the final result.
- `analyst`: combines repository investigation, impact analysis, and implementation planning.
- `implementer`: handles code, tests, configuration, and documentation changes within one approved scope.
- `verifier`: runs applicable project checks and reports exact evidence without editing.
- `reviewer`: performs final correctness, regression, maintainability, security, and dependency review.

The workflow intentionally combines closely related responsibilities:

- Investigation and planning belong together in `analyst`.
- Multi-file changes, localized fixes, and documentation changes belong together in `implementer`; the leader defines the scope guard.
- Security review belongs with `reviewer` because it is part of final change risk assessment.

## Structure

```text
.
├── agents.template.md    # starter template for target projects
├── agents/
│   ├── analyst.md
│   ├── implementer.md
│   ├── leader.md
│   ├── reviewer.md
│   └── verifier.md
├── opencode.json
└── LICENSE
```

## Configuration

Defined in [`opencode.json`](./opencode.json):

- Enabled provider: `opencode-go` only.
- Default agent: `leader`.
- Maximum subagent nesting depth: `1`.
- Project instruction file: `AGENTS.md`, used when present in the opened project.
- Built-in `plan` and `explore` remain available; built-in `build` and `general` are disabled.
- Global and custom agent model settings use only the approved configured model families.

When the configured primary model or provider is unavailable, activate a fallback through OpenCode's normal model/provider override or the provider's documented operational procedure, including any required authentication. No unsupported fallback-specific configuration key is used here.

## Per-project context

OpenCode resolves `AGENTS.md` against the project being opened, not this global configuration repository. Agents use an existing project `AGENTS.md` as the project's operational source of truth. If no project `AGENTS.md` exists, suggest running the `/init` command to generate one.

## Workflow

1. `leader` establishes repository context and classifies the request. For medium-to-large projects, independent read-only analysis may run in parallel with disjoint questions and file sets.
2. `analyst` investigates and plans non-trivial, unfamiliar, broad, or risky work.
3. `implementer` performs the approved mutation.
4. `verifier` runs the applicable checks after every mutation.
5. `reviewer` reviews every code or configuration mutation, including security and dependency risk when relevant.
6. `leader` inspects final Git status and diff before reporting completion. Implementation has one owner and mutation/verification gates remain serialized.

Trivial, obvious changes may skip `analyst` only when the leader states why. Failed, missing, or ambiguous verification blocks completion.

## Permissions

Permissions are defined per custom agent so the custom workflow's safety policy does not leak into OpenCode's built-in `plan` and `explore` agents. Secret-file reads are denied for every custom agent. Only `implementer` may edit files; `leader`, `analyst`, `verifier`, and `reviewer` are read-only. Bash is deny-by-default for `leader` and `analyst`, while `implementer` and `verifier` use approval-gated `*: ask` with a narrow read-only Git allowlist and `git *: deny`. `leader` may delegate to the four specialists; `analyst` and `reviewer` may not. `external_directory` is denied for all custom agents.

The installed SDK's permission type formally covers `read`, `edit`, `glob`, `grep`, `list`, `bash`, `task`, `external_directory`, `todowrite`, `question`, `webfetch`, `websearch`, `lsp`, `doom_loop`, and `skill`. Runtime per-key enforcement should still be confirmed against the running OpenCode version.

The leader carries a bounded Context block between stages: it retains exact file references and acceptance criteria while summarizing verified, decision-relevant findings and replacing superseded details. It does not accumulate whole files or unbounded command output.

Restart OpenCode after changing this configuration because agents and configuration load at startup.

## Installation

Clone this repository into OpenCode's global configuration directory:

```bash
git clone https://github.com/joaortcoelho/opencode-config.git ~/.config/opencode
```

Restart OpenCode after changing this configuration because agents and configuration load at startup.

## License

See [`LICENSE`](./LICENSE).
