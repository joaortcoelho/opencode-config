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

Permissions are defined in two layers:

- The top-level `permission` block in [`opencode.json`](./opencode.json) contains the shared read-only, edit-deny, deny-by-default safety, LSP, and narrow Git policy.
- Global safety defaults explicitly deny editing, task delegation, and access outside the configured project directory; Bash is denied by default.
- Per-agent permission blocks are explicit exceptions to the global defaults and contain only intentional overrides, such as delegation, editing, command approval, web access, and todo/question behavior.

OpenCode merges agent permissions with the global configuration, with agent rules taking precedence. Bash permissions use command-pattern matching; the explicit read-only Git allowlist contains only `git status`, `git status --short`, `git diff`, `git diff --stat`, `git diff --name-only`, `git log`, and `git log --oneline -10`. Implementer and verifier additionally have a broad `"git *": deny` rule with an explicit safe Git allowlist for those commands plus `pwd`, `git ls-files`, and `git diff --check`; Linux-specific command denies were removed in favor of approval-gated `*: ask` for other shell commands.

The built-in `plan` and `explore` agents remain available and are read-only under the global policy. Invoking them directly does not run the custom analyst/verifier/reviewer workflow gates.

The installed SDK's permission type formally covers `read`, `edit`, `glob`, `grep`, `list`, `bash`, `task`, `external_directory`, `todowrite`, `question`, `webfetch`, `websearch`, `lsp`, `doom_loop`, and `skill`. Runtime merge behavior and per-key enforcement should still be confirmed against the running OpenCode version.

The leader and read-only agents cannot edit files. The leader can inspect Git state and delegate only to approved specialists. The implementer may edit files, but mutation commands and non-allowlisted shell commands require approval. The verifier requires approval for project checks except the small universal read-only allowlist. Project-specific approved commands belong in an explicit `AGENTS.md` section and still require and obey the applicable approval semantics; dynamic `AGENTS.md` commands are intentionally not encoded into JSON permissions. The reviewer is read-only and performs the final permission and security review.

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
