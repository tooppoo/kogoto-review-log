# `revg review`

Manage review sessions, review runs, findings, folding, and session end.

## `revg review start`

Start a worktree-local review session.

```bash
revg review start --base main
```

`--base` is required.

Options:

```text
--base <revision>
  Required. Git base revision for the whole review session.

-c, --config <path>
  Config file to use.
```

The base revision is stable for the session.

The review cursor is stored separately as `last_reviewed_head`.

## `revg review status`

Show active session status.

```bash
revg review status
```

Example output:

```text
active session: yes
session id: revg-session-20260622-001
base: main
head: HEAD
last reviewed head: abc123
findings: 5
unresolved: 2
criteria proposals: 1
```

## `revg review run`

Run one AI review pass in the active session.

```bash
revg review run
```

Default scope is `incremental`.

```bash
revg review run --scope incremental
```

Full review:

```bash
revg review run --scope full
```

Options:

```text
--scope <incremental|full>
  Review scope.

--format <json|toon|markdown|text>
  Temporary view output format.

--no-view
  Do not generate a human-readable temporary view.

--strict
  Fail on invalid records, ambiguous session state, or invalid config.

--dry-run
  Build the review context and planned agent request without saving findings.

-c, --config <path>
  Config file to use.
```

Scope behavior:

```text
incremental
  Focus on last_reviewed_head..HEAD.
  Include base..HEAD summary context, unresolved findings, and relevant criteria.

full
  Review base..HEAD as a whole.
  Intended before human review, PR creation, or session end.
```

There is no special final review scope.

A full review may still produce findings. If it does, the session should continue.

`review run` performs:

```text
load active session
  -> compute review scope
  -> load relevant criteria
  -> load unresolved findings
  -> generate review context
  -> invoke the configured single agent
  -> extract JSON
  -> validate schema
  -> retry if schema-invalid
  -> save valid findings
  -> generate temporary view
  -> update last_reviewed_head
```

Schema retry behavior is configured through `schema_retry.max_attempts`. See [Configuration](../configuration.md).

## `revg review show`

Show findings in the active session.

```bash
revg review show
```

Show only unresolved findings:

```bash
revg review show --unresolved
```

Options:

```text
--unresolved
  Show only unresolved findings.

--format <json|toon|markdown|text>
  Output format.

-c, --config <path>
  Config file to use.
```

## `revg review resolve`

Mark a finding as resolved by a specific resolution state.

```bash
revg review resolve <review-id> --as fixed
```

Options:

```text
--as <state>
  Required resolution state.

--reason <text>
  Reason for the resolution.

-c, --config <path>
  Config file to use.
```

Suggested resolution states:

```text
fixed
rejected
deferred
accepted-negative
superseded
duplicate
```

Examples:

```bash
revg review resolve finding-20260622-001 --as fixed
```

```bash
revg review resolve finding-20260622-002 \
  --as accepted-negative \
  --reason "Compatibility concern is intentionally not applied to this project stage."
```

## `revg review fold`

Create criteria update proposals from findings in the active session.

```bash
revg review fold
```

`fold` does not directly update criteria.

It writes proposal JSON, for example:

```text
.reviewography/proposals/proposal-20260622-001.json
```

Options:

```text
--format <json|toon|markdown|text>
  Temporary view output format.

-c, --config <path>
  Config file to use.
```

## `revg review end`

End the active review session.

```bash
revg review end
```

Ending a session should remove or clear the active session pointer.

It should not delete canonical findings, run records, or criteria proposals.

If unresolved findings remain, the command should warn.

Options:

```text
--force
  End the session even if unresolved findings remain.

-c, --config <path>
  Config file to use.
```
