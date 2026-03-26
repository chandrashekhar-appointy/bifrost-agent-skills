# Troubleshooting

Prefer CLI-first diagnosis before escalating to raw API, Kubernetes, or infrastructure debugging.

## Auth Failures

Start with:

```bash
bifrost auth whoami --json --non-interactive
```

If unauthenticated:
- use PAT for automation
- use `--headless` for browserless human sessions
- use browser login for local desktop sessions

## Missing `.bifrost.yaml`

If deploy or binding context is missing, create or refresh it with:

```bash
bifrost init ... --json --non-interactive
```

## Missing Git Remote

If the repo has no valid remote, do not force Bifrost to compensate. Follow `git-prep-workflow.md`.

## PAT vs Device vs Browser Confusion

Select auth mode by environment:
- PAT for agents and CI
- device flow for headless humans
- browser login for local desktops

## Monorepo Ambiguity

If multiple services or Dockerfiles could apply, stop and resolve:
- root directory
- Dockerfile path
- Docker context

Do not guess.

## Infra Provisioning Errors

Start with:

```bash
bifrost infra list --json --non-interactive
bifrost infra get <resource-id> --json --non-interactive
```

Check whether the resource exists, whether it is ready, and whether binding was attempted.

## Build Failure vs Build Timeout

Use:

```bash
bifrost build get <build-id> --json --non-interactive
bifrost build wait <build-id> --json --non-interactive
```

Treat timeout and failure as different cases:
- failure means the build reached a terminal error
- timeout means the build did not reach a healthy terminal state in time

## Deployment Timeout or Unhealthy Rollout

Use:

```bash
bifrost deployment get <deployment-id> --json --non-interactive
bifrost deployment wait <deployment-id> --json --non-interactive
```

Prefer these commands before reaching for lower-level platform debugging.
