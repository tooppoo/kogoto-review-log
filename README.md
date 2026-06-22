# reviewography

`reviewography` is a local-first review memory tool for AI-assisted software development.

It records review findings as structured, AI-readable data, generalizes them into reusable review knowledge, and uses that knowledge when reviewing future changes.

The name combines `review` and `-graphy`: writing, describing, and organizing review knowledge. It is not a visualization tool.

## Purpose

AI-assisted development often produces useful review findings, but those findings are easily lost.

Common problems include:

* review findings are copied manually between tools
* the same class of issue is rediscovered repeatedly
* accepted, rejected, and out-of-scope review comments are not preserved as reusable knowledge
* human reviewers must inspect large diffs and large test suites without enough structured context
* AI agents lack a stable local memory of past review decisions

`reviewography` addresses these problems by treating review findings as persistent project knowledge.

## Core idea

A review finding should not disappear after a single review round.

Instead, reviewography turns each review into structured data:

```text
current change
  ↓
relevant past review records
  ↓
review
  ↓
new review findings
  ↓
structured review record
  ↓
future review context
```

The canonical output is not a Markdown report.
The canonical output is AI-readable review data.

Human-readable Markdown is a temporary view generated from that data.

## Goals

reviewography aims to:

* record review findings in a structured format
* distinguish concrete findings from generalized review patterns
* preserve accepted, rejected, deferred, and negative review decisions
* reuse past review knowledge in future changes
* help AI agents and humans review changes with better context
* operate primarily on local files
* remain independent from any specific implementation or review loop tool

## Non-goals

reviewography does not aim to:

* execute implementation tasks
* run an implementation-review-fix loop
* replace Claude Code, Codex, ChatGPT, or other AI tools
* manage GitHub Issues or Pull Requests
* create ADRs
* perform full repository architecture audits
* analyze test coverage by itself
* become a single integrated development automation platform

Implementation and review loops should be handled by skills, scripts, or external tools.

Repository-wide structure audits should be handled by a separate tool.

Test evidence extraction should be handled by a separate tool such as `testography`.

## Positioning

reviewography is an independent tool.

It may be used together with tools such as:

* `kogoto` for issue refinement and development workflow support
* `testography` for test evidence mapping
* repository audit tools for documentation and implementation consistency checks
* AI coding agents for implementation
* AI review agents for review

But reviewography does not depend on those tools.

The intended dependency direction is:

```text
other workflow tools may call reviewography
reviewography does not depend on those workflow tools
```

## Primary workflow

The basic workflow is one step:

```text
run reviewography against a change
  → load relevant prior review records
  → review the current change
  → record new review findings
  → output structured review data and temporary human-readable views
```

Example:

```bash
reviewography review --base main --head HEAD
```

or with the short alias:

```bash
revg review --base main --head HEAD
```

## Review records

reviewography distinguishes several concepts.

### Review run

A review run represents one execution of reviewography against a change.

It records:

* base revision
* head revision
* changed files
* relevant historical review records
* new findings
* generated temporary views
* tool metadata

### Review finding

A review finding is a concrete issue found in a specific review.

Example:

```json
{
  "artifact_type": "review_finding",
  "schema_version": "0",
  "id": "finding-20260622-001",
  "category": "test-gap",
  "severity": "medium",
  "target": {
    "kind": "module",
    "path": "cmd/tools/run"
  },
  "claim": "Hidden command visibility is not directly tested through user-facing help output.",
  "evidence": [
    {
      "kind": "file",
      "path": "cmd/tools/run_test.go",
      "note": "No test asserts that the hidden command is excluded from normal help output."
    }
  ],
  "recommended_action": "Add a help output test for hidden command visibility.",
  "decision": "proposed"
}
```

### Review pattern

A review pattern is a generalized lesson derived from one or more findings.

Example:

```json
{
  "artifact_type": "review_pattern",
  "schema_version": "0",
  "id": "pattern-hidden-command-help-visibility",
  "title": "Hidden command visibility should be tested through user-facing help output",
  "category": "test-gap",
  "applies_when": [
    "A CLI command is hidden.",
    "Command visibility is part of the expected user-facing behavior.",
    "Help output is used as the discovery surface."
  ],
  "heuristic": "Do not verify hidden command behavior only by implementation flags. At least one user-facing help output test should confirm whether the command is shown or hidden.",
  "priority": "medium",
  "status": "accepted",
  "occurrences": [
    {
      "finding_id": "finding-20260622-001",
      "issue": "51"
    }
  ]
}
```

### Review decision

A review decision records how a finding or pattern was handled.

Supported decision states include:

* `proposed`
* `accepted`
* `rejected`
* `deferred`
* `needs-human-decision`
* `accepted-negative`
* `superseded`

`accepted-negative` is used when a rejected concern becomes an explicit future policy.

Example:

```json
{
  "artifact_type": "review_pattern",
  "schema_version": "0",
  "id": "pattern-pre-v0-breaking-change-policy",
  "title": "Do not block pre-v0 changes solely for backward compatibility",
  "category": "scope-policy",
  "heuristic": "Before v0, conceptual consistency and simplicity may take precedence over backward compatibility.",
  "status": "accepted-negative"
}
```

This is important because review memory should preserve not only what to care about, but also what not to overemphasize.

## Data-first design

reviewography prioritizes AI-readable records.

Human-readable reports are generated views.

```text
canonical:
  review records
  findings
  patterns
  decisions

temporary:
  Markdown summaries
  terminal output
  review packets
  human-facing checklists
```

Temporary views may be regenerated and should not be treated as the source of truth.

## Local-first storage

A typical project may store reviewography data under:

```text
.reviewography/
  config.json
  records/
    findings/
    patterns/
    decisions/
  runs/
    <run-id>/
      review-run.json
      findings.json
      relevant-patterns.json
      view.md
```

The exact storage layout may evolve, but the principle is stable:

* structured data is canonical
* local files are the primary storage
* generated human-readable views are secondary

Projects may choose whether to commit selected review records.

A conservative default is:

```text
commit:
  shared review patterns
  explicit project review policies

do not commit by default:
  raw AI review output
  temporary views
  run-local metadata
  sensitive findings
```

## Review philosophy

reviewography does not assume every AI review comment is correct.

Review findings may be:

* valid defects
* test gaps
* design questions
* scope expansions
* policy conflicts
* duplicated concerns
* incorrect AI interpretations

Therefore, reviewography records both the finding and its decision state.

The purpose is not to accumulate warnings blindly.
The purpose is to build a reusable, inspectable review memory.

## Relationship to human review

reviewography does not remove human review.

It changes the unit of review.

Instead of asking humans to inspect every changed line first, it helps them inspect:

* relevant prior review patterns
* current findings
* evidence for each finding
* unresolved risks
* repeated issue classes
* rejected or out-of-scope concerns

Human review remains responsible for final judgment.

## Status

reviewography is initially intended as a small local CLI.

The first useful version should focus on:

1. loading relevant prior review records
2. reviewing a change once
3. recording new findings as structured data
4. generating temporary human-readable output

Implementation-review loops, repository audits, and test evidence extraction are intentionally out of scope for the core tool.
