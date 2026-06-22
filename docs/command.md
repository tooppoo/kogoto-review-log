# Command index

This document is an index for reviewography commands.

Detailed command references are split by command group.

## Common command name

Canonical command:

```bash
reviewography
```

Short command:

```bash
revg
```

`revg` should behave identically to `reviewography`.

## Common option

Most commands accept:

```bash
-c, --config <path>
```

This selects a project-local configuration file explicitly.

See [Configuration](configuration.md) for lookup order and config format.

## Command groups

| Command group | Reference | Purpose |
|---|---|---|
| `revg init` | [docs/command/init.md](command/init.md) | Initialize reviewography files for a repository. |
| `revg review` | [docs/command/review.md](command/review.md) | Manage review sessions, runs, findings, folding, and session end. |
| `revg criteria` | [docs/command/criteria.md](command/criteria.md) | List, show, search, edit, weight, validate, export, and apply criteria. |
| `revg tools` | [docs/command/tools.md](command/tools.md) | Install helper material for external tools. |

## Suggested workflow commands

```bash
revg init
revg review start --base main
revg review run
revg review show --unresolved
revg review resolve <review-id> --as fixed
revg review run --scope full
revg review fold
revg criteria apply --local <proposal-id>
revg review end
```

See [Workflow](workflow.md) for the full usage flow.

## Scope boundaries

reviewography commands cover:

- worktree-local session management
- single-agent review execution
- schema validation and retry
- finding persistence
- explicit finding resolution
- criteria proposal generation and application
- criteria management
- temporary human-readable views

reviewography commands do not cover:

- implementation changes
- automatic fix loops
- multi-agent orchestration
- finding merge or deduplication across agents
- GitHub Pull Request creation or merge
- GitHub PR comment ingestion
- ADR creation
- repository-wide audits
- test evidence extraction
- user-level config
