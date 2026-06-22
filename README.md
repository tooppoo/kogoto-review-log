# reviewography

[![CI](https://github.com/tooppoo/reviewography/actions/workflows/ci.yml/badge.svg)](https://github.com/tooppoo/reviewography/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/tooppoo/reviewography/graph/badge.svg?token=KdgB8qZyUk)](https://codecov.io/gh/tooppoo/reviewography)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

`reviewography` is a local-first review memory tool for AI-assisted software development.

It runs AI reviews against local changes, records review findings as structured data, and turns useful findings into reusable review criteria.

The short command name is:

```bash
revg
```

## What reviewography does

reviewography helps keep review knowledge from being lost between AI-assisted development sessions.

It focuses on:

- worktree-local review sessions
- AI-readable review findings
- reusable review criteria
- schema-validated AI review output
- explicit finding resolution
- criteria proposals derived from review sessions

## What reviewography does not do

reviewography does not own the whole development workflow.

It does not:

- implement code changes
- run a fix-review loop until findings disappear
- orchestrate multi-agent review
- merge or deduplicate findings from multiple agents
- replace human review
- manage GitHub Issues or Pull Requests
- create ADRs
- perform repository-wide audits
- extract test evidence from source code

Those concerns should be handled by other tools, skills, scripts, or future integrations.

## Documentation

- [Workflow](docs/workflow.md): how reviewography is used during a review session.
- [Model](docs/model.md): core concepts such as sessions, runs, findings, criteria, proposals, and review cursor.
- [Configuration](docs/configuration.md): project-local configuration, agent adapter settings, schema retry, and config lookup.
- [Command index](docs/command.md): index of CLI commands.

Command references:

- [`revg init`](docs/command/init.md)
- [`revg review`](docs/command/review.md)
- [`revg criteria`](docs/command/criteria.md)
- [`revg tools`](docs/command/tools.md)

## Basic usage

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

See [Workflow](docs/workflow.md) for details.

## Design principles

1. Structured data is canonical.
2. Human-readable output is a generated view.
3. Review sessions are local to the active worktree.
4. One review run uses one configured review agent.
5. Invalid AI output must not become canonical review data.
6. Review findings are concrete; criteria are reusable.
7. Updating criteria is explicit and proposal-based.
