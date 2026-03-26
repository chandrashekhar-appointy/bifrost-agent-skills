# Repo Context

Bifrost stores repo-local deployment context in `.bifrost.yaml`.

## When to Run `bifrost init`

Run `bifrost init` when:
- `.bifrost.yaml` is missing
- the repo is being attached to a different project or service
- root, Dockerfile, context, branch, or repo URL settings need to be refreshed

Example:

```bash
bifrost init --project <project> --service <service> --environment <env> --json --non-interactive
```

## Fields That Matter Most

These fields are the main context anchors:
- `project_id`
- `service_id`
- `environment_id`
- `repository_url`
- `default_branch`
- `root_directory`
- `dockerfile_path`
- `docker_context`

## Resolution Rules

- explicit flags override `.bifrost.yaml`
- `.bifrost.yaml` overrides global defaults
- if `.bifrost.yaml` is needed and absent, create or update it through `bifrost init`

## Fresh vs Existing Repo Decision

- existing `.bifrost.yaml` usually means this repo is already linked to a Bifrost app; prefer reuse unless the user wants to relink
- no `.bifrost.yaml` on a fresh repo usually means create a new project and new service by default
- do not silently attach a fresh repo to an unrelated existing project just because one exists in the account

## Practical Guidance

Before deploy or infra binding:
- check whether `.bifrost.yaml` already provides enough context
- avoid asking the user for IDs if the CLI can resolve them through existing repo context
- avoid rewriting `.bifrost.yaml` unless the repo really needs new linkage or refreshed build settings
