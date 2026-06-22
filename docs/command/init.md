# `revg init`

Initialize reviewography files for a repository.

```bash
revg init
```

## Purpose

`revg init` creates the local directory structure and default project configuration used by reviewography.

## Expected files

Example layout:

```text
.reviewography/
  criteria/
  proposals/
  records/
    findings/
    runs/
  views/
  tmp/
reviewography.config.kdl
```

## Options

```text
--force
  Overwrite existing local configuration.

--minimal
  Create only the minimum required files.

--gitignore
  Add recommended run-local paths to .gitignore.

-c, --config <path>
  Use a specific config file path.
```

## Recommended `.gitignore`

```gitignore
.reviewography/session.json
.reviewography/records/runs/
.reviewography/views/
.reviewography/tmp/
```

Repository-level criteria may be committed when they define shared review behavior.

## Notes

`revg init` does not start a review session.

Use `revg review start --base <revision>` after initialization.
