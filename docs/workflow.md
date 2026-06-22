# reviewography workflow

This document describes the intended workflow for `reviewography`.

reviewography is designed around one core operation:

```text
review a change once,
using prior review memory,
then record the new review result.
```

It does not run an implementation-review-fix loop.

## Core workflow

The basic flow is:

```text
1. Start from a current change
2. Run reviewography
3. Load relevant prior review records
4. Review the current change
5. Record new findings
6. Generate structured output
7. Optionally generate temporary human-readable views
```

In command form:

```bash
reviewography review --base main --head HEAD
```

or:

```bash
revg review --base main --head HEAD
```

## Step 1: Start from a change

A change is usually identified by a Git base and head.

Examples:

```bash
reviewography review --base main --head HEAD
reviewography review --base origin/main --head feature/review-memory
reviewography review --base v0.1.0 --head HEAD
```

reviewography inspects the change and builds a local review context.

The context may include:

* changed files
* changed modules
* diff summary
* existing review records
* relevant review patterns
* optional external review input
* optional test evidence input

## Step 2: Load relevant prior review records

Before reviewing the current change, reviewography searches existing review memory.

It looks for review patterns relevant to the current change.

Matching may use:

* changed file paths
* module names
* command names
* previous finding categories
* project-specific review policies
* explicit tags
* issue or ADR text when provided

Example matched patterns:

```text
- Hidden command visibility should be tested through user-facing help output
- Output format changes should preserve structured result semantics
- Before v0, conceptual consistency may take precedence over backward compatibility
```

These patterns are used as review context.

They are not automatically treated as findings.

## Step 3: Review the current change

reviewography performs one review pass against the current change.

The review should consider:

* the current diff
* relevant prior review patterns
* project review policies
* supplied issue or ADR context
* optional test evidence
* optional external review comments

The result is a set of concrete findings.

A finding should include:

* category
* severity
* target
* claim
* evidence
* recommended action
* decision state

Example categories:

```text
implementation-defect
test-gap
spec-doc-mismatch
design-inconsistency
scope-creep
risk
redundancy
ai-readability
review-policy
```

## Step 4: Record findings

New findings are recorded as structured data.

The canonical record is JSON or another machine-readable structured format.

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
  "claim": "The new output format flag is not tested together with command-level flags.",
  "evidence": [
    {
      "kind": "diff",
      "path": "cmd/root.go",
      "note": "The output format flag is parsed globally."
    },
    {
      "kind": "file",
      "path": "cmd/root_test.go",
      "note": "Tests cover the flag alone but not interaction with command-level flags."
    }
  ],
  "recommended_action": "Add an integration test covering global output flags together with command-specific flags.",
  "decision": "proposed"
}
```

## Step 5: Generate output

reviewography writes structured records first.

Example output layout:

```text
.reviewography/
  runs/
    20260622-001/
      review-run.json
      relevant-patterns.json
      findings.json
      view.md
```

The structured files are canonical.

The Markdown file is a temporary view.

## Temporary human-readable view

A human-readable view may be generated for convenience.

Example:

```md
# Reviewography Review

## Change
- base: main
- head: HEAD

## Relevant prior patterns
1. Hidden command visibility should be tested through user-facing help output
2. Output format changes should preserve structured result semantics

## Findings
### finding-20260622-001
- category: test-gap
- severity: medium
- target: cmd/root_test.go
- claim: The new output format flag is not tested together with command-level flags.
- recommended action: Add an integration test covering global output flags together with command-specific flags.

## No finding
No implementation defect was detected in the reviewed diff.

## Notes
This Markdown file is a generated view. The structured review records are canonical.
```

The Markdown view should not be edited as the source of truth.

If a human wants to change the result, they should update the structured decision record through reviewography commands.

## Decision workflow

After reviewography records findings, a human or external workflow may decide them.

Decision states:

```text
proposed
accepted
rejected
deferred
needs-human-decision
accepted-negative
superseded
```

Example:

```bash
reviewography decide finding-20260622-001 --decision accepted
```

or:

```bash
reviewography decide finding-20260622-002 \
  --decision accepted-negative \
  --reason "Before v0, backward compatibility should not block simplification."
```

## Generalization workflow

A concrete finding may be generalized into a reusable review pattern.

Example:

```bash
reviewography generalize finding-20260622-001
```

Concrete finding:

```text
cmd/tools/run is hidden, but help output does not have a direct test.
```

Generalized pattern:

```text
Hidden command visibility should be tested through user-facing help output.
```

The generalized pattern can be used in future reviews.

## Reuse workflow

On future changes, reviewography loads relevant patterns before reviewing.

Example:

```bash
reviewography review --base main --head HEAD
```

During review, it may detect that a prior pattern is relevant:

```text
Relevant prior pattern:
- Hidden command visibility should be tested through user-facing help output

Current change:
- Adds or modifies hidden CLI command behavior

Review consequence:
- Check whether user-facing help output behavior is tested.
```

If the current change satisfies the pattern, no finding is needed.

If it violates the pattern, a new finding is recorded.

## Handling no findings

A review run may produce no findings.

That is still useful.

The run should record:

* the change reviewed
* relevant prior patterns considered
* reviewer metadata
* conclusion that no finding was produced

Example:

```json
{
  "artifact_type": "review_run",
  "schema_version": "0",
  "id": "run-20260622-001",
  "base": "main",
  "head": "HEAD",
  "relevant_patterns": [
    "pattern-hidden-command-help-visibility"
  ],
  "findings": [],
  "conclusion": "no-findings"
}
```

A no-finding run is not proof that the change is correct.
It is only a record of one review pass.

## Out of scope

The following are outside this workflow:

* automatically asking an implementation agent to fix findings
* repeatedly running a reviewer until no findings remain
* deciding whether to create a PR
* merging changes
* managing GitHub Issues
* creating ADRs
* performing full repository audits
* extracting test evidence from test code

Those may be handled by other tools or skills.

reviewography's responsibility is limited to:

```text
review memory in
review of current change
review memory out
```

## Recommended integration

A larger AI-assisted development workflow may use reviewography like this:

```text
1. Discuss design in ChatGPT
2. Record conclusion and ADR plan in an Issue
3. Implement with an AI coding agent
4. Run reviewography once against the change
5. Use reviewography findings in a separate implementation-review loop if needed
6. Generate or update human-readable review views
7. Perform human review
8. Merge
```

reviewography participates in the review step, but does not own the entire workflow.
