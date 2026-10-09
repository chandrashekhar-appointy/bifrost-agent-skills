# Runtime Diagnosis

Use this after a build succeeds but the deployment does not become healthy.

## Stop-and-Diagnose Rule

Once a build for the target commit has succeeded:
- do not create another deployment immediately
- switch to diagnosis first

## Diagnostic Order

0. whether the service is stopped. A stopped service (scaled to 0) has no
   pods, so its URL answers 503 and it has no new logs; that is not a failure.

```bash
bifrost scale <service> --json --non-interactive
```

   If the environment's `state` is `stopped`, tell the user. Start it before
   debugging only if they want it running:
   `bifrost scale <service> --env <env> --replicas 1 --json --non-interactive`.

1. deployment status

```bash
bifrost deployment get <deployment-id> --json --non-interactive
bifrost deployment wait <deployment-id> --json --non-interactive
```

2. build status, only to confirm the image is not the blocker

```bash
bifrost build get <build-id> --json --non-interactive
```

3. if rollout still fails, read the service's runtime logs. They cover every
   replica, including pods that crashed and restarted, and are kept for 3 days:

```bash
bifrost logs <service> --env <env> --since 15m --tail 200 --json --non-interactive
bifrost logs <service> --env <env> --since 15m --grep ERROR --json --non-interactive
```

   then the rest of the platform/runtime evidence:
- deployment describe
- service port mapping
- route status

## Classification Rules

- `bifrost scale` shows `stopped`:
  - stopped by the owner, not a failure; start it if the user wants it running
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
