# Deploy Workflow

Use this workflow for app delivery through the Bifrost CLI.

## Preflight Decision Tree

1. Verify auth.

```bash
bifrost auth whoami --json --non-interactive
```

2. Inspect repo state.
- check whether `.git` exists
- check whether a remote already exists
- check whether `.bifrost.yaml` exists
- inspect the app shape: framework, Dockerfile, likely port, likely root directory, likely monolith vs microservice
- choose the matching framework reference before writing or changing Dockerfiles:
  - Next.js: `framework-nextjs.md`
  - static Vite/nginx: `framework-vite-static.md`
  - long-running Node service: `framework-node-service.md`
- if Dockerfile changes are needed, follow `dockerfile-guidance.md`

3. Decide whether this is an existing linked app or a fresh deploy.
- existing `.bifrost.yaml` with matching repo: default to existing project/service/environment
- fresh repo with no `.bifrost.yaml`: default to a new project and a new service
- do not silently reuse an unrelated existing project just because one exists

4. Ask one compact batch of questions only if critical choices remain ambiguous.
Examples:
- create a new GitHub repo or use an existing remote
- repo visibility if a new GitHub repo must be created
- which app/root to deploy when the repo is a monorepo
- whether the app needs new infra or already uses existing infra

## Canonical Flow

1. Ensure the repository has a valid remote. If not, follow `git-prep-workflow.md`.
2. Ensure the code intended for deploy is committed and pushed.
3. Ensure `.bifrost.yaml` exists and matches the intended project/service/environment.

```bash
bifrost init --project <project> --service <service> --environment <env> --json --non-interactive
```

4. Deploy.

```bash
bifrost deploy --wait --json --non-interactive
```

Before issuing a manual deploy after a push, check whether the same commit already has an active webhook deployment:

```bash
bifrost deployment latest --project <project> --environment <env> --commit <sha> --source-type webhook --json --non-interactive
```

If a webhook deployment for that commit is already pending or deploying, wait on it instead of creating a duplicate manual deployment.

This is the default single-run behavior:
- push once
- wait on the webhook deployment if it already exists
- only create a manual deployment when webhook deploy is absent or intentionally not being used

## App Inference Rules

### Project Type

Use valid project types only:
- `monolith`
- `microservice`

Heuristic:
- single app at repo root or one deployable service: `monolith`
- clearly separate deployable services or multi-app layout: `microservice`

### Port and Runtime Shape

Infer the likely runtime port before creating or updating the service.
Use evidence from:
- `Dockerfile` `EXPOSE`
- framework defaults such as Next.js on `3000`
- app start command
- documented health endpoints

Do not assume port `80` for every app.

### Env and Infra Signals

Inspect:
- `.env`, `.env.local`, `.env.production`, `.env.example`
- DB/cache/pub-sub libraries and env variable names

If infra already looks externalized, do not auto-create new infra.
If the app clearly needs infra and nothing is configured, ask once whether to use existing infra or create new Bifrost-managed infra.

## Supported Variants

### Existing `.bifrost.yaml`

If `.bifrost.yaml` already contains the correct project, service, repo URL, and build context, prefer:

```bash
bifrost deploy --wait --json --non-interactive
```

### Fresh Repo With Existing Remote

Create a new project/service by default, initialize `.bifrost.yaml`, then deploy.

### Fresh Repo With No Remote

Bootstrap git/remote first. Do not ask `bifrost` to create the repository.

### Monorepo Deploy

When repo layout is ambiguous, explicitly supply:
- root directory
- Dockerfile path
- Docker context

Do not guess monorepo paths if multiple valid candidates exist.

## Waiting and Diagnostics

Do not implement custom polling. Use:

```bash
bifrost build wait <build-id> --json --non-interactive
bifrost deployment wait <deployment-id> --json --non-interactive
```

Use `bifrost deploy --wait` when a single command can own the full wait lifecycle.

If the build succeeds but the deployment fails, stop and diagnose the runtime before creating a new deployment. Do not keep triggering more deployments for the same commit until you know why the existing one failed.

Use `runtime-diagnosis.md` for the concrete stop-and-diagnose sequence.

## Post-Deploy Operations

When service env vars or secrets change, prefer an explicit config apply instead of asking the user to redeploy the image:

```bash
bifrost service apply-config <service> --project <project> --environment <env> --json --non-interactive
```

When the runtime needs an operational bounce without rebuilding:

```bash
bifrost deployment restart <deployment-id> --json --non-interactive
```

Use restart for operational recovery. Use config apply when config changed and the running Helm release needs to be reconciled.

## Success Output

After a successful deploy, always report:
- deployment status
- project
- environment
- service
- public URL
- deployment ID
- build ID when available
