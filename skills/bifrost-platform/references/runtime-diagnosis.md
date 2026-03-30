# Runtime Diagnosis

Use this after a build succeeds but the deployment does not become healthy.

## Stop-and-Diagnose Rule

Once a build for the target commit has succeeded:
- do not create another deployment immediately
- switch to diagnosis first

## Diagnostic Order

1. deployment status

```bash
bifrost deployment get <deployment-id> --json --non-interactive
bifrost deployment wait <deployment-id> --json --non-interactive
```

2. build status, only to confirm the image is not the blocker

```bash
bifrost build get <build-id> --json --non-interactive
```

3. if rollout still fails, move to platform/runtime evidence:
- pod logs
- deployment describe
- service port mapping
- route status

## Classification Rules

- build succeeded + pod crashlooping:
  - runtime/container problem
- build succeeded + pods healthy + no route:
  - routing trigger/platform problem
- build succeeded + route exists + URL not live yet:
  - routing in progress / edge propagation
- build succeeded + root URL works + one feature route fails:
  - app config issue, not total deployment failure

## Practical Guidance

- if the URL is not live immediately after route creation, report `routing in progress`
- only report `edge ready` after repeated healthy public responses
- if the same commit already has an active webhook deployment, wait on it instead of creating a manual duplicate
