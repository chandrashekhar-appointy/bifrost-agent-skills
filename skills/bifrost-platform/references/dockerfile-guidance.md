# Dockerfile Guidance

Use this reference when a deploy task needs Dockerfile review, creation, or repair.

## Decision Rule

- If the repo already has a Dockerfile that matches the app framework and runtime, prefer fixing it over replacing it.
- Only create a new Dockerfile when:
  - none exists
  - the existing one is clearly incompatible with the app
  - the existing one fails in a known way and cannot be repaired cleanly

## One-Pass Safety Rules

- infer the runtime type first:
  - Next.js app server
  - static Vite/nginx site
  - long-running Node service
- infer the runtime port before touching service config
- prefer non-root containers, but make runtime write paths explicit
- avoid Dockerfile steps that mutate mounted Kubernetes paths such as `/var/run/secrets/...`
- be careful with broad recursive `chown` commands in Kaniko builds

## Nginx / Static Site Rules

For nginx-based static containers:
- do not assume nginx temp/cache paths are writable for a non-root user
- prefer temp paths under `/tmp`
- ensure the process listens on an unprivileged port such as `8080`
- ensure the service port and container port match the chosen runtime port

If nginx crashes with errors like:
- `mkdir() \"/var/cache/nginx/client_temp\" failed (13: Permission denied)`

then fix runtime write paths before redeploying.

## Next.js Rules

- default runtime port is usually `3000`
- ensure the container listens on `0.0.0.0`
- do not default service routing to `80` unless the container really serves on `80`

## Node Service Rules

- default runtime port is often `3000`, `4000`, or `8080`
- prefer an explicit health endpoint when the app provides one
- if none exists, use `/` only when the app reliably serves it

## When to Stop and Diagnose

If the build succeeds but rollout fails:
- do not write another Dockerfile immediately
- inspect:
  - container logs
  - deployment describe
  - service port mapping
  - route status

Fix the concrete runtime failure instead of repeatedly rebuilding.
