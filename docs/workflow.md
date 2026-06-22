# reviewography workflow

This document describes the intended workflow for `reviewography` v0.

reviewography is designed around one core operation:

```text
run one AI review pass in an active review session,
using prior review criteria,
then record schema-valid review findings.
```

It does not run an implementation-review-fix loop.

## Core workflow

The normal flow is:

```text
1. Initialize reviewography in the repository
2. Start a worktree-local review session with a required base revision
3. Make or update implementation changes outside reviewography
4. Run one AI review pass
5. Inspect unresolved findings
6. Resolve findings after external fixes or decisions
7. Repeat change and review as needed
8. Run a full review before ending the session
9. Fold session findings into criteria proposals
10. Explicitly apply selected proposals
11. End the review session
```

In command form:

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

## Step 1: Initialize the repository

A repository should be initialized before review sessions are used.

```bash
revg init
```

The initial layout is local-first.

Example:

```text
.reviewography/
  criteria/
  proposals/
  records/
  views/
  tmp/
reviewography.config.kdl
```

The exact layout may evolve, but structured JSON data remains canonical.

`reviewography.config.kdl` is project-local. v0 does not use user-level config.

A different config file may be specified explicitly:

```bash
revg review run -c reviewography.codex.kdl
```

Configuration lookup order:

```text
1. -c / --config
2. ./reviewography.config.kdl
3. built-in defaults
```

## Step 2: Start a review session

A review session is started with a required base revision.

```bash
revg review start --base main
```

The session is intended to be worktree-local.

When used with Git worktrees, each worktree should have its own active session file at the worktree root. This lets each worktree maintain an independent review cursor.

A session records at least:

```json
{
  "session_id": "revg-session-20260622-001",
  "base": "main",
  "current_head": "HEAD",
  "last_reviewed_head": null
}
```

`base` defines the whole change under review.

`last_reviewed_head` is the review cursor used by incremental review.

## Step 3: Make changes outside reviewography

reviewography does not implement code changes.

Implementation may be done by a human, Claude Code, Codex, another AI coding agent, or any external workflow.

reviewography only manages the review side:

```text
change exists
  -> reviewography reviews it
  -> findings are recorded
  -> external workflow fixes or rejects findings
```

## Step 4: Run an incremental review

Run:

```bash
revg review run
```

This defaults to:

```bash
revg review run --scope incremental
```

An incremental review focuses on:

```text
last_reviewed_head..HEAD
```

It still includes:

- the session base
- base..HEAD summary context
- relevant criteria
- unresolved findings
- previous run metadata

This prevents each review from losing the whole-session context while avoiding unnecessary full-diff review on every iteration.

## Step 5: AI reviewer execution

`revg review run` invokes one configured AI review agent.

The command performs:

```text
active session lookup
  -> review context generation
  -> criteria lookup
  -> agent invocation through adapter
  -> JSON extraction
  -> schema validation
  -> retry on schema-invalid output
  -> valid finding persistence
  -> temporary view generation
  -> review cursor update
```

Agent configuration is read from the selected config file.

Example:

```kdl
reviewography {
  review {
    default_scope "incremental"

    schema_retry {
      max_attempts 3
    }

    agent {
      provider "codex"
      model "gpt-5.5-codex"
      effort "high"
    }
  }
}
```

`provider` is required.

`model`, `subagent`, and `effort` are provider-specific and optional depending on the selected provider adapter.

## Step 6: Schema retry

AI review output must validate before it becomes canonical review data.

If the AI output is not valid JSON, or if the JSON does not satisfy the review finding schema, reviewography asks the same agent to regenerate the output.

`max_attempts` includes the initial generation.

```text
max_attempts = 1
  initial generation only

max_attempts = 3
  initial generation + 2 regenerations
```

v0 constraints:

```text
default: 3
minimum: 1
maximum: 10
```

If all attempts fail, the review run fails and no invalid finding is saved as canonical data.

Invalid raw output may be saved only as run-local debug data.

## Step 7: Inspect findings

Show findings from the active session:

```bash
revg review show
```

Show only unresolved findings:

```bash
revg review show --unresolved
```

Findings are concrete review comments produced during a session.

Example:

```json
{
  "artifact_type": "review_finding",
  "schema_version": "0",
  "id": "finding-20260622-001",
  "category": "test-gap",
  "severity": "medium",
  "target": {
    "kind": "file",
    "path": "cmd/root_test.go"
  },
  "claim": "The output format flag is not tested together with command-level flags.",
  "evidence": [
    {
      "kind": "diff",
      "path": "cmd/root.go",
      "note": "The output format flag is parsed globally."
    }
  ],
  "recommended_action": "Add an integration test for global output flags combined with command-specific flags.",
  "resolution": "unresolved"
}
```

## Step 8: Resolve findings

Findings are resolved explicitly.

```bash
revg review resolve <review-id> --as fixed
```

Resolution states should distinguish different outcomes.

Suggested v0 states:

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
  --reason "Before v0, backward compatibility should not block simplification."
```

`accepted-negative` records that a rejected concern is itself useful future review knowledge.

## Step 9: Run a full review

Before human review, PR creation, or session end, run a full review:

```bash
revg review run --scope full
```

A full review examines:

```text
base..HEAD
```

There is no special `final` scope.

A full review can still produce findings. If it does, the session should continue until the findings are resolved, deferred, or otherwise decided.

## Step 10: Fold findings into criteria proposals

`revg review fold` summarizes session findings into criteria update proposals.

```bash
revg review fold
```

`fold` does not directly update criteria.

It creates proposal JSON.

Example:

```text
.reviewography/proposals/proposal-20260622-001.json
```

A concrete finding such as:

```text
cmd/tools/run is hidden, but help output does not have a direct test.
```

may produce a proposal such as:

```text
Hidden command visibility should be tested through help output.
```

The proposal is still only a proposal. It must be explicitly applied.

## Step 11: Apply criteria proposals

Apply a proposal to repository-level criteria:

```bash
revg criteria apply --local <proposal-id>
```

Apply a proposal to user-level criteria:

```bash
revg criteria apply --global <proposal-id>
```

v0 excludes user-level config, but user-level criteria may still be used as a separate criteria store.

Because user-level criteria are environment-dependent and interact poorly with devcontainers or CI reproducibility, repository-level criteria should be preferred for review behavior that must be shared or reproducible.

## Step 12: Edit criteria

Criteria are canonical JSON records, but humans should not edit the JSON directly.

v0 supports editing only the title or body through an editor buffer.

```bash
revg criteria edit <criteria-id>
revg criteria edit --body <criteria-id>
revg criteria edit --title <criteria-id>
```

`revg criteria edit <criteria-id>` is equivalent to:

```bash
revg criteria edit --body <criteria-id>
```

The tool updates the relevant JSON field after the editor exits and then validates the result.

## Step 13: End the session

End the active session:

```bash
revg review end
```

Ending a session should remove or clear the active session pointer, but it should not delete canonical findings, run records, or criteria proposals.

If unresolved findings remain, `review end` should warn.

## Multi-agent review

v0 does not orchestrate multi-agent review.

One config means one review agent.

If multi-agent review is needed, run separate processes with separate config files:

```bash
revg review run -c reviewography.codex.kdl
revg review run -c reviewography.claude.kdl
```

reviewography v0 does not merge, deduplicate, or reconcile those findings automatically.

## GitHub PR review comments

Human manual review is expected to happen through GitHub PR comments or reviews.

Importing PR comments into reviewography is outside v0.

That capability should be designed together with a future GitHub Action that can translate PR comments into structured reviewography findings.

## Temporary human-readable views

Human-readable output is generated from structured records.

It is a temporary view, not the source of truth.

Example:

```md
# Reviewography Review

## Session
- base: main
- head: HEAD
- scope: incremental

## Criteria used
1. Hidden command visibility should be tested through help output
2. Output format changes should preserve structured result semantics

## Findings
### finding-20260622-001
- category: test-gap
- severity: medium
- target: cmd/root_test.go
- claim: The output format flag is not tested together with command-level flags.
- resolution: unresolved
```

If a human wants to change review state, they should use reviewography commands so that structured data remains canonical.

## Recommended integration

A larger AI-assisted development workflow may use reviewography like this:

```text
1. Discuss design in ChatGPT
2. Record conclusion and ADR plan in an Issue
3. Implement with an AI coding agent
4. Run revg review run
5. Use findings in an external implementation-review loop if needed
6. Resolve findings explicitly
7. Run revg review run --scope full
8. Fold findings into criteria proposals
9. Apply selected criteria proposals
10. Perform human PR review
11. Merge
```

reviewography participates in the review step, but it does not own the entire development workflow.

## Out of scope

The following are outside the v0 workflow:

- automatically fixing findings
- repeatedly running implementation and review until findings disappear
- multi-agent orchestration
- merging or deduplicating multi-agent findings
- deciding whether to create a PR
- merging changes
- managing GitHub Issues
- creating ADRs
- performing full repository audits
- extracting test evidence from test code
- importing GitHub PR comments

Those may be handled by other tools, skills, scripts, or future integrations.
