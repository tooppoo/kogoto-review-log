# `revg criteria`

Manage reusable review criteria.

Criteria are canonical JSON records. Markdown, text, and TOON outputs are generated views.

## `revg criteria list`

List criteria IDs, titles, weights, and status.

```bash
revg criteria list
```

Options:

```text
--format <json|toon|markdown|text>
  Output format.

-c, --config <path>
  Config file to use.
```

## `revg criteria show`

Show criteria.

```bash
revg criteria show --format markdown
revg criteria show --format json
revg criteria show --format toon
```

Options:

```text
--format <json|toon|markdown|text>
  Output format.

--id <criteria-id>
  Show only one criterion.

-c, --config <path>
  Config file to use.
```

`--format` is the canonical form.

Short flags such as `--json` or `--toon` may be added later as aliases, but command handling should normalize to `--format`.

## `revg criteria search`

Search criteria.

```bash
revg criteria search "hidden command"
```

Options:

```text
--category <category>
  Filter by category.

--path <path>
  Search criteria relevant to a path.

--format <json|toon|markdown|text>
  Output format.

-c, --config <path>
  Config file to use.
```

## `revg criteria weight up`

Increase criteria weight.

```bash
revg criteria weight up <criteria-id>
revg criteria weight up <criteria-id> 2
```

The default amount is `1`.

`weight` means review priority. It should not be used as occurrence count.

## `revg criteria weight down`

Decrease criteria weight.

```bash
revg criteria weight down <criteria-id>
revg criteria weight down <criteria-id> 2
```

The default amount is `1`.

## `revg criteria edit`

Edit a criteria field through an editor buffer.

```bash
revg criteria edit <criteria-id>
revg criteria edit --body <criteria-id>
revg criteria edit --title <criteria-id>
```

`revg criteria edit <criteria-id>` is equivalent to:

```bash
revg criteria edit --body <criteria-id>
```

Supported editable fields:

```text
title
body
```

The command should not expose the full JSON record for direct editing.

It should:

```text
1. read the criteria JSON
2. extract the selected field
3. open the field content in $EDITOR
4. write the updated field back to JSON
5. validate the updated criteria record
```

## `revg criteria deprecate`

Mark criteria as deprecated without deleting its history.

```bash
revg criteria deprecate <criteria-id> --reason "covered by broader CLI public surface criterion"
```

## `revg criteria apply`

Apply a criteria proposal.

```bash
revg criteria apply --local <proposal-id>
revg criteria apply --global <proposal-id>
```

Options:

```text
--local <proposal-id>
  Apply the proposal to repository-level criteria.

--global <proposal-id>
  Apply the proposal to user-level criteria.

-c, --config <path>
  Config file to use.
```

Repository-level criteria should be preferred when review behavior must be reproducible.

User-level criteria are environment-dependent.

## `revg criteria export`

Export criteria.

```bash
revg criteria export --format json --out criteria.json
revg criteria export --format markdown --out criteria.md
```

Options:

```text
--format <json|toon|markdown|text>
  Export format.

--out <path>
  Output path.

--scope <local|global|all>
  Criteria scope to export.

-c, --config <path>
  Config file to use.
```

There is no dedicated `criteria import` command.

A user may place exported criteria files in the expected criteria location and then run validation.

## `revg criteria validate`

Validate criteria records.

```bash
revg criteria validate
```

Validation should check:

- required fields
- artifact type
- schema version
- unique IDs
- valid status
- valid weight
- valid proposal references
- malformed body/title content

Options:

```text
--strict
  Treat warnings as errors.

--format <json|toon|markdown|text>
  Output format.

-c, --config <path>
  Config file to use.
```
