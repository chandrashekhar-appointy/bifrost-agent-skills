---
name: bifrost-platform
description: Use this skill when an agent needs to deploy applications, manage infrastructure, authenticate, or troubleshoot delivery workflows through the Bifrost CLI. It teaches the agent to use the native `bifrost` binary directly, run a strong preflight for repo/app/platform context, prefer `git` before `gh`, pick sensible project/service defaults, and finish with deployment URLs and actionable diagnostics.
---

# Bifrost Platform

Use this skill when the task involves deploying an app with Bifrost, creating or binding Bifrost-managed infra, initializing repo context with `.bifrost.yaml`, or diagnosing build and deployment failures through the Bifrost CLI.

## Core Rules

- Use the native `bifrost` binary directly.
- If the `bifrost` binary is not installed, stop and provide the install command before continuing. Do not assume the skill can install it automatically.
- Append `--json --non-interactive` to every meaningful `bifrost` command used for workflow execution.
- Treat CLI JSON on stdout as the source of truth. Do not parse human-formatted output when JSON is available.
- Never invent project IDs, service IDs, environment IDs, deployment IDs, build IDs, or infra IDs when `.bifrost.yaml` or CLI lookup can resolve them.
- Never deploy to production unless the user explicitly asks for production.
- Never pass `file://` URLs or local absolute filesystem paths as Bifrost deploy sources.
- Prefer `git` for repo inspection and existing remotes. Use `gh` only when a new GitHub repo must be created from this machine or when GitHub-specific visibility/permission changes are required.
- Prefer CLI wait commands such as `bifrost build wait` and `bifrost deployment wait` instead of implementing your own polling loop.

## Preflight First

Before making changes, inspect the repo and platform context.

Check:
- whether `.git` exists and whether a remote already exists
- whether `.bifrost.yaml` already links the repo to a project/service/environment
- the app shape: framework, Dockerfile, root directory, build context, likely runtime port, likely monolith vs microservice
- env files and infra signals such as `DATABASE_URL`, Prisma, Redis, pub/sub clients, or `.env.example`
- whether the task is clearly a fresh app deploy or a change to an existing linked app

If confidence is high, proceed without asking.
If key choices are still ambiguous, ask one compact batch of questions instead of drip-feeding prompts across the workflow.

## Default Decisions

- Fresh repo with no `.bifrost.yaml`: default to a new project and new service.
- Existing `.bifrost.yaml`: default to the linked project/service unless the user says to relink.
- Single app repo: default to `monolith`.
- Multiple independently deployable apps: use `microservice` only when the repo structure clearly supports it.
- Existing git remote: reuse it instead of creating a new GitHub repo.
- New GitHub repo creation: ask once for visibility and owner/name if that cannot be inferred.

## Final Output Contract

After a successful deployment, always return:
- deployment status
- project
- environment
- service
- public application URL
- deployment ID
- build ID when available

If deploy fails, classify the failure before suggesting next steps:
- repo/bootstrap issue
- GitHub auth or GitHub App permission issue
- app build failure
- runtime/health failure
- platform rollout failure

## Workflow Map

- For command shape, JSON rules, context resolution, and wait behavior, read `references/cli-contract.md`.
- For choosing between PAT, headless device flow, and browser login, read `references/auth-modes.md`.
- For `.bifrost.yaml` expectations and when to run `bifrost init`, read `references/repo-context.md`.
- For local repos, existing remotes, and optional `gh` usage, read `references/git-prep-workflow.md`.
- For end-to-end deploy execution, including fresh-vs-existing project decisions and post-deploy URL reporting, read `references/deploy-workflow.md`.
- For database and bucket creation or binding, including when to ask whether infra is needed at all, read `references/infra-workflow.md`.
- For auth, repo, infra, build, and rollout failures, read `references/troubleshooting.md`.

## Operating Stance

Keep the agent thin and procedural:
- use `bifrost` for Bifrost actions
- use `git` for local repo work and existing remotes
- use `gh` only for GitHub-specific repo creation or permission changes when needed
- avoid committing agent-local metadata such as `.agents/`, `.claude/`, `.codex/`, editor folders, or raw `.env*` files unless the user explicitly wants them committed
- be deliberate about committing `.bifrost.yaml`; it is often useful, but it is still a repo-level choice
- use CLI-native wait commands before escalating to raw API or cluster debugging
