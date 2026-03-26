# Repo Context

Bifrost stores repo-local deployment context in `.bifrost.yaml`.

## When to Run `bifrost init`

Run `bifrost init` when:
- `.bifrost.yaml` is missing
- the repo is being attached to a different project or service
- root, Dockerfile, or context settings need to be refreshed

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
- `root_directory`
- `dockerfile_path`
- `docker_context`

## Resolution Rules

- explicit flags override `.bifrost.yaml`
- `.bifrost.yaml` overrides global defaults
- if `.bifrost.yaml` is needed and absent, create or update it through `bifrost init`

## Practical Guidance

Before deploy or infra binding:
- check whether `.bifrost.yaml` already provides enough context
- avoid asking the user for IDs if the CLI can resolve them through existing repo context
