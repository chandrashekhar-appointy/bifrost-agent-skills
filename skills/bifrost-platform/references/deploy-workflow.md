# Deploy Workflow

Use this workflow for app delivery through the Bifrost CLI.

## Canonical Flow

1. Verify auth.

```bash
bifrost auth whoami --json --non-interactive
```

2. Ensure the repository has a valid remote. If not, follow `git-prep-workflow.md`.
3. Ensure `.bifrost.yaml` exists and matches the intended project/service/environment.

```bash
bifrost init --project <project> --service <service> --environment <env> --json --non-interactive
```

4. Deploy.

```bash
bifrost deploy --wait --json --non-interactive
```

## Supported Variants

### Remote Repo Deploy With Existing `.bifrost.yaml`

If `.bifrost.yaml` already contains the correct project, service, repo URL, and build context, prefer:

```bash
bifrost deploy --wait --json --non-interactive
```

### Remote Repo Deploy Without `.bifrost.yaml`

Initialize context first, then deploy.

### Local Repo With Existing Remote

Validate that the intended branch and remote are pushed, then use `bifrost init` if context is missing or stale.

### Local Repo With No Remote

Use `git` plus `gh` first. Do not ask `bifrost` to create the repository.

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
