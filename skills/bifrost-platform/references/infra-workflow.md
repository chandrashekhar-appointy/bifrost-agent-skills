# Infra Workflow

Use Bifrost infra commands to create shared resources and bind them to services.

## Supported Infra in v1

- database
- bucket

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

## Defaults and Expectations

- when infra is project-linked and environment is unspecified, default to `dev`
- standalone infra may omit project linkage
- after binding, verify the returned metadata before moving on to deploy
- if the user wants both infra and deploy, finish binding first and then run deploy
