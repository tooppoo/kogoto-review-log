# `revg tools`

Install helper material for external tools.

## `revg tools install`

Install an integration helper.

```bash
revg tools install <tool-name>
```

Initial tool names:

```text
codex-skill
claude-skill
```

Examples:

```bash
revg tools install codex-skill
revg tools install claude-skill
```

These helpers should teach the target AI environment how to use reviewography.

They should not replace reviewography's own schema validation and retry mechanism.

## Scope

`revg tools install` may create files such as skills, prompts, or local helper instructions for another AI environment.

It should not:

- run review sessions
- modify criteria directly
- resolve findings directly
- run implementation-review loops
- bypass schema validation
