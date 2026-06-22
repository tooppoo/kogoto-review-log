# Configuration

reviewography uses a project-local configuration file.

The default path is:

```text
reviewography.config.kdl
```

A different config file can be selected with:

```bash
-c, --config <path>
```

Example:

```bash
revg review run -c reviewography.codex.kdl
```

## Lookup order

```text
1. -c / --config
2. ./reviewography.config.kdl
3. built-in defaults
```

User-level config is not supported.

User-level criteria may still exist as a criteria store, but configuration itself is project-local or explicitly selected by `--config`.

## Example

```kdl
reviewography {
  review {
    default_scope "incremental"

    schema_retry {
      max_attempts 3
    }

    agent {
      provider "codex"
      model "example-review-model"
      effort "high"
    }
  }

  criteria {
    local ".reviewography/criteria"
    global "~/.config/reviewography/criteria"
  }

  output {
    default_format "markdown"
  }
}
```

## Review settings

### `default_scope`

Default review scope for `revg review run`.

Allowed values:

```text
incremental
full
```

If omitted, the default is `incremental`.

### `schema_retry.max_attempts`

Maximum number of agent generation attempts when schema validation fails.

`max_attempts` includes the initial generation.

```text
max_attempts = 1
  initial generation only

max_attempts = 3
  initial generation + 2 regenerations
```

Constraints:

```text
default: 3
minimum: 1
maximum: 10
```

Schema-invalid output is not saved as canonical review data.

## Agent settings

One config defines one active review agent.

```kdl
agent {
  provider "codex"
  model "example-review-model"
  effort "high"
}
```

### `provider`

Required. The provider selects the agent adapter.

### `model`

Provider-specific. Some providers may require it. Others may ignore it.

### `subagent`

Optional and provider-specific.

### `effort`

Optional and provider-specific.

## Single-agent configuration

reviewography does not orchestrate multi-agent review.

To run multiple agents, create multiple config files and run reviewography separately:

```bash
revg review run -c reviewography.codex.kdl
revg review run -c reviewography.claude.kdl
```

The resulting findings are not merged or deduplicated automatically.

## Criteria settings

```kdl
criteria {
  local ".reviewography/criteria"
  global "~/.config/reviewography/criteria"
}
```

`local` is the repository-level criteria path.

`global` is the user-level criteria path.

Repository-level criteria should be preferred when review behavior must be reproducible across machines, devcontainers, or CI.

## Output settings

```kdl
output {
  default_format "markdown"
}
```

Supported output formats:

```text
json
toon
markdown
text
```

Structured JSON remains canonical regardless of temporary view format.
