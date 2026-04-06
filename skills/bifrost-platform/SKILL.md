---
name: bifrost-platform
description: Use this skill when an agent needs to deploy applications, manage infrastructure, authenticate, or troubleshoot delivery workflows through the Bifrost CLI. It teaches the agent to use the native `bifrost` binary directly, run a strong preflight for repo/app/platform context, prefer `git` before `gh`, pick sensible project/service defaults, and finish with deployment URLs and actionable diagnostics.
---

# Bifrost Platform

Use this skill when the task involves deploying an app with Bifrost, creating or binding Bifrost-managed infra, initializing repo context with `.bifrost.yaml`, or diagnosing build and deployment failures through the Bifrost CLI.

## Core Rules

- Use the native `bifrost` binary directly.
- If the `bifrost` binary is not installed, install it with `npm install -g @chandrashekharchoudha/bifrost-cli` before continuing.
- Append `--json --non-interactive` to every meaningful `bifrost` command used for workflow execution.
- Treat CLI JSON on stdout as the source of truth. Do not parse human-formatted output when JSON is available.
- Never invent project IDs, service IDs, environment IDs, deployment IDs, build IDs, or infra IDs when `.bifrost.yaml` or CLI lookup can resolve them.
- Never deploy to production unless the user explicitly asks for production.
- Never pass `file://` URLs or local absolute filesystem paths as Bifrost deploy sources.
- Prefer `git` for repo inspection and existing remotes. Use `gh` only when a new GitHub repo must be created from this machine or when GitHub-specific visibility/permission changes are required.
- Prefer CLI wait commands such as `bifrost build wait` and `bifrost deployment wait` instead of implementing your own polling loop.
- Optimize for a single clean deployment pass:
  - prepare once
  - push once
  - wait once
  - diagnose before retrying

## Preflight First

Before making changes, inspect the repo and platform context.

Check:
- whether `.git` exists and whether a remote already exists
- whether `.bifrost.yaml` already links the repo to a project/service/environment
- the app shape: framework, Dockerfile, root directory, build context, likely runtime port, likely monolith vs microservice
- whether the app is a Next.js app, static Vite/nginx app, or a long-running Node service
- env files and infra signals such as `DATABASE_URL`, Prisma, Redis, pub/sub clients, or `.env.example`
- whether push-triggered webhook deploys are already active for the repo/service
- whether the task is clearly a fresh app deploy or a change to an existing linked app
- for private GitHub repos or freshly created repos, whether the repo is already covered by a GitHub App installation that Bifrost can use for short-lived clone tokens

If confidence is high, proceed without asking.
If key choices are still ambiguous, ask one compact batch of questions instead of drip-feeding prompts across the workflow.

## Default Decisions

- Fresh repo with no `.bifrost.yaml`: default to a new project and new service.
- Existing `.bifrost.yaml`: default to the linked project/service unless the user says to relink.
- Static site repo with no server/runtime signals: default to `static_site`.
- If the user already has a prebuilt container image or explicitly wants to skip builds, prefer an image-backed container service with `source_type=image` over inventing a repository build flow.
- Single long-running app repo: default to `monolith`.
- Multiple independently deployable apps: use `microservice` only when the repo structure clearly supports it.
- Existing git remote: reuse it instead of creating a new GitHub repo.
- New GitHub repo creation: ask once for visibility and owner/name if that cannot be inferred.
- Templates are an abstraction over ordinary Bifrost project + service creation. Treat template launches as image-backed service deployments with optional infra and runtime env/secrets layered on top.
- For repository-backed deploys on private GitHub repos, assume Bifrost should clone with a short-lived GitHub App installation token. If that token path is not available for the repo, stop and fix GitHub App access before blaming the app build.

### Framework Routing Rules

- Vite static sites published on Bifrost static hosting must use relative or otherwise subpath-safe asset paths. Prefer `base: './'` unless the repo already has a correct static base strategy.
- Next.js with `output: 'export'` and no server-only or backend/runtime signals should be treated as a `static_site`. For Bifrost v1 path-based hosting, exported apps need an appropriate `basePath` that matches `/s/<project-slug>`.
- Next.js with `output: 'standalone'` plus dynamic behavior such as server actions, backend routes, auth callbacks, database calls, or other runtime dependencies should be treated as a containerized application, not a static site.
- If the user selected static hosting but the Next.js repo is configured with `output: 'standalone'` and the app is otherwise static, fix the repo to `output: 'export'` before deploying it as a static site.
- If the project metadata already exists, prefer `project.type` as the top-level deployment mode and `service.kind` as the runtime behavior check.

## Final Output Contract

After a successful deployment, always return:
- deployment status
- whether routing is still in progress or edge is ready
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

## Single-Run Contract

Default to this sequence:

1. inspect repo and platform context once
2. ask one compact batch of questions only if a critical decision cannot be inferred
3. create or update project/service/environment context once
4. commit and push once
5. if a new private GitHub repo was created or selected, verify that the repo is covered by the user’s GitHub App installation before deploying
6. if the GitHub App is not installed at all, tell the user to install it first
7. if the GitHub App is installed for selected repositories only and the new repo is missing, tell the user to add that repo to the installation and then sync it
8. check whether a webhook deployment already exists for the pushed commit
9. wait on that deployment instead of creating a duplicate manual deployment
10. if the build fails, fix the build
11. if the build succeeds but the deployment fails, switch to runtime diagnosis before triggering another deployment
12. after success, always return the URL and the final status

## Workflow Map

- For command shape, JSON rules, context resolution, and wait behavior, read `references/cli-contract.md`.
- For choosing between PAT, headless device flow, and browser login, read `references/auth-modes.md`.
- For `.bifrost.yaml` expectations and when to run `bifrost init`, read `references/repo-context.md`.
- For local repos, existing remotes, and optional `gh` usage, read `references/git-prep-workflow.md`.
- For end-to-end deploy execution, including fresh-vs-existing project decisions and post-deploy URL reporting, read `references/deploy-workflow.md`.
- For Dockerfile decisions and framework-specific container guidance, read `references/dockerfile-guidance.md`.
- For Next.js deploy expectations, read `references/framework-nextjs.md`.
- For Vite/static site deploy expectations, read `references/framework-vite-static.md`.
- For the first-class static-site hosting path, read `references/static-site-hosting.md`.
- For direct image-backed deploys and image-backed templates, read `references/image-and-template-deploy.md`.
- For long-running Node service deploy expectations, read `references/framework-node-service.md`.
- For database and bucket creation or binding, including when to ask whether infra is needed at all, read `references/infra-workflow.md`.
- For runtime diagnosis after a successful build but failed rollout, read `references/runtime-diagnosis.md`.
- For auth, repo, infra, build, and rollout failures, read `references/troubleshooting.md`.

## Operating Stance

Keep the agent thin and procedural:
- use `bifrost` for Bifrost actions
- use `git` for local repo work and existing remotes
- use `gh` only for GitHub-specific repo creation or permission changes when needed
- avoid committing agent-local metadata such as `.agents/`, `.claude/`, `.codex/`, editor folders, or raw `.env*` files unless the user explicitly wants them committed
- be deliberate about committing `.bifrost.yaml`; it is often useful, but it is still a repo-level choice
- use CLI-native wait commands before escalating to raw API or cluster debugging
- after a push, check whether a webhook deployment already exists for the same commit before creating a manual deployment
- if a build succeeds but the deployment fails, stop creating more deployments and switch to diagnosis first
- if a deployment is healthy but the public URL is not live yet, report `routing in progress` until edge probes succeed
