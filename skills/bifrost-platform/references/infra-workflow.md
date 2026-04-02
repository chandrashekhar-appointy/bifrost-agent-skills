# Infra Workflow

Use Bifrost infra commands to create shared resources and bind them to services.

## Supported Infra in v1

- database
- bucket
- pubsub
- redis

## Infra Intent First

Before creating anything, inspect whether the app already uses external infra.

Signals include:
- `.env*` entries such as `DATABASE_URL`, `REDIS_URL`, `POSTGRES_*`, `MYSQL_*`, `MONGO_*`, `PUBSUB_*`
- libraries such as Prisma, Sequelize, `pg`, Redis clients, pub/sub SDKs
- existing service env/secrets already configured in Bifrost

Decision rule:
- if the app already appears wired to existing infra, do not auto-create new infra
- if the app clearly needs infra and nothing is configured, ask one focused question: use existing infra or create new Bifrost-managed infra
- if the app does not appear to need infra, skip infra creation entirely

## Create Database

```bash
bifrost infra create database <name> --project <project> --environment <env> --json --non-interactive
```

## Create Bucket

```bash
bifrost infra create bucket <name> --project <project> --environment <env> --json --non-interactive
```

## Create Pub/Sub

```bash
bifrost infra create pubsub <name> --project <project> --environment <env> --json --non-interactive
```

## Create Redis

```bash
bifrost infra create redis <name> --project <project> --environment <env> --json --non-interactive
```

## List and Inspect

```bash
bifrost infra list --json --non-interactive
bifrost infra get <resource-id> --json --non-interactive
```

## Bind to Service

Binding should be explicit, not assumed.

```bash
bifrost infra bind <resource-id> --service <service> --json --non-interactive
```

## Delete Infra

Delete is supported, but only for the user who originally created the infra resource.

Do not attempt delete while a resource is still bound to a service.

```bash
bifrost infra delete <resource-id> --json --non-interactive
```

## Defaults and Expectations

- when infra is project-linked and environment is unspecified, default to `dev`
- standalone infra may omit project linkage
- `redis` is a shared platform instance with per-resource logical allocations, not a dedicated Redis server per app
- shared Redis currently uses one platform auth secret plus a unique `REDIS_KEY_PREFIX` per resource for logical isolation
- backend must be configured with `SHARED_REDIS_INSTANCE`, `SHARED_REDIS_HOST`, `SHARED_REDIS_PORT`, `SHARED_REDIS_PASSWORD`, `SHARED_REDIS_DATABASE`, and `SHARED_REDIS_SCHEME` before Redis create can succeed
- `pubsub` creates a shared platform topic resource
- Pub/Sub bind now creates a real GCP Pub/Sub subscription per service/environment binding and exposes both `PUBSUB_SUBSCRIPTION` and `PUBSUB_SUBSCRIPTION_PATH`
- cluster rollout for Pub/Sub requires the `XPubSub` XRD/composition plus RBAC allowing the workflow executor to manage `xpubsubs` and managed `subscriptions.pubsub.gcp.m.upbound.io`
- after binding, verify the returned metadata before moving on to deploy
- if the user wants both infra and deploy, finish binding first and then run deploy
