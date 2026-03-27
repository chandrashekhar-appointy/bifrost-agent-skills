# CLI Contract

## Required Invocation Pattern

Use the installed `bifrost` binary directly and append:

```bash
--json --non-interactive
```

Example:

```bash
bifrost project list --json --non-interactive
```

Use this default for all meaningful workflow commands, including auth checks, init, infra, deploy, build, deployment, and diagnosis commands.

Operational follow-up commands also use the same contract:

```bash
bifrost service apply-config <service> --project <project> --environment <env> --json --non-interactive
bifrost deployment restart <deployment-id> --json --non-interactive
bifrost deployment list --project <project> --environment <env> --commit <sha> --json --non-interactive
bifrost deployment latest --project <project> --environment <env> --commit <sha> --source-type webhook --json --non-interactive
```

## Output Rules

- Stdout is the parse target.
- With `--json`, stdout should contain machine-readable JSON only.
- Stderr may contain warnings, progress, browser/device prompts, or human commentary.
- Prefer JSON fields over text fragments.

## Context Resolution Order

Resolve command context in this order:

1. explicit flags on the current command
2. `.bifrost.yaml` in the repository
3. global CLI config in `~/.bifrost/config.json`
4. fail with actionable guidance

Do not hallucinate IDs when the CLI can resolve them.

## Wait Behavior

Use CLI-native wait commands instead of agent-side polling loops.

Preferred commands:

```bash
bifrost build wait <build-id> --json --non-interactive
bifrost deployment wait <deployment-id> --json --non-interactive
bifrost deploy --wait --json --non-interactive
```

## Safe Defaults

- default environment is `dev` unless explicitly set otherwise
- production requires explicit user intent
- if required context is missing, stop and surface the missing field rather than guessing
