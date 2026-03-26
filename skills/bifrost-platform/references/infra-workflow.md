# Infra Workflow

Use Bifrost infra commands to create shared resources and bind them to services.

## Supported Infra in v1

- database
- bucket

## Create Database

```bash
bifrost infra create database --name <name> --project <project> --environment <env> --json --non-interactive
```

## Create Bucket

```bash
bifrost infra create bucket --name <name> --project <project> --environment <env> --json --non-interactive
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
