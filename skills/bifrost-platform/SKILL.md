---
name: bifrost-platform
description: Use this skill when an agent needs to deploy applications, manage infrastructure, authenticate, or troubleshoot delivery workflows through the Bifrost CLI. It teaches the agent to use the native `bifrost` binary directly, rely on JSON output, resolve local repo context from `.bifrost.yaml`, and use `git` plus `gh` for remote bootstrap before calling Bifrost.
---

# Bifrost Platform

Use this skill when the task involves deploying an app with Bifrost, creating or binding Bifrost-managed infra, initializing repo context with `.bifrost.yaml`, or diagnosing build and deployment failures through the Bifrost CLI.

## Core Rules

- Use the native `bifrost` binary directly.
- Append `--json --non-interactive` to every meaningful `bifrost` command used for workflow execution.
- Treat CLI JSON on stdout as the source of truth. Do not parse human-formatted output when JSON is available.
- Never invent project IDs, service IDs, environment IDs, deployment IDs, build IDs, or infra IDs when `.bifrost.yaml` or CLI lookup can resolve them.
- Never deploy to production unless the user explicitly asks for production.
- Never pass `file://` URLs or local absolute filesystem paths as Bifrost deploy sources.
- If the repo has no valid remote, use `git` and `gh` first, then `bifrost init`, then `bifrost deploy`.
- Prefer CLI wait commands such as `bifrost build wait` and `bifrost deployment wait` instead of implementing your own polling loop.

## Workflow Map

- For command shape, JSON rules, context resolution, and wait behavior, read `references/cli-contract.md`.
- For choosing between PAT, headless device flow, and browser login, read `references/auth-modes.md`.
- For `.bifrost.yaml` expectations and when to run `bifrost init`, read `references/repo-context.md`.
- For local repos that do not yet have a remote, read `references/git-prep-workflow.md`.
- For end-to-end deploy execution, including monorepos, read `references/deploy-workflow.md`.
- For database and bucket creation or binding, read `references/infra-workflow.md`.
- For auth, repo, infra, build, and rollout failures, read `references/troubleshooting.md`.

## Operating Stance

Keep the agent thin and procedural:
- use `bifrost` for Bifrost actions
- use `git` and `gh` for Git hosting/bootstrap
- use explicit lookups before assumptions
- use CLI-native wait commands before escalating to raw API or cluster debugging
