# Troubleshooting

Prefer CLI-first diagnosis before escalating to raw API, Kubernetes, or infrastructure debugging.

## Missing `bifrost` Binary

If `bifrost` is not installed, the skill does not make that happen automatically by itself.

First provide or run the documented install path, then continue:

```bash
npm install -g @chandrashekharchoudha/bifrost-cli
```

Use the npm install path for the Bifrost CLI. Do not switch to a Homebrew install path for `bifrost`.

## Auth Failures

Start with:

```bash
bifrost auth whoami --json --non-interactive
```

If unauthenticated:
- use PAT for automation
- use `--headless` for browserless human sessions
- use browser login for local desktop sessions

## Missing `.bifrost.yaml`

If deploy or binding context is missing, create or refresh it with:

```bash
bifrost init ... --json --non-interactive
```

## Missing Git Remote

If the repo has no valid remote, do not force Bifrost to compensate. Follow `git-prep-workflow.md`.

## PAT vs Device vs Browser Confusion

Select auth mode by environment:
- PAT for agents and CI
- device flow for headless humans
- browser login for local desktops

## GitHub App and Private Repo Access

Treat private repo deploy as supported, but verify access carefully.

Check:

```bash
bifrost github installations --json --non-interactive
bifrost github repos <installation-id> --json --non-interactive
```

If the repo is missing from the installation list:
- it is a GitHub App permission problem, not an app build problem
- if the user chose selected-repository installation mode, tell them to add the repo to the GitHub App installation
- if no installation exists at all, tell them to install the GitHub App first
- sync the installation after access is granted

```bash
bifrost github sync <installation-id> --json --non-interactive
```

If the repo appears in the installation list but clone still fails, classify that separately as a platform/private-clone issue.

For repository-backed builds, remember the intended mechanism:
- backend resolves the repo to a GitHub App installation
- backend generates a short-lived installation token through the internal GitHub endpoint
- build/static-publish workflow uses that token for clone

If workflow logs say:

```text
No GitHub installation configured for <repo>; falling back to anonymous git clone
```

then the token path did not engage for that repo. Treat that as a repo-to-installation mapping problem first.

If push automation is enabled, check whether a webhook deployment for the same commit already exists before creating a manual deployment:

```bash
bifrost deployment latest --project <project> --environment <env> --commit <sha> --source-type webhook --json --non-interactive
```

## Monorepo Ambiguity

If multiple services or Dockerfiles could apply, stop and resolve:
- root directory
- Dockerfile path
- Docker context

Do not guess.

## Infra Provisioning Errors

Start with:

```bash
bifrost infra list --json --non-interactive
bifrost infra get <resource-id> --json --non-interactive
```

Check whether the resource exists, whether it is ready, and whether binding was attempted.

## Build Failure vs Build Timeout

Use:

```bash
bifrost build get <build-id> --json --non-interactive
bifrost build logs <build-id> --json --non-interactive
bifrost build wait <build-id> --json --non-interactive
```

Treat timeout and failure as different cases:
- failure means the build reached a terminal error
- timeout means the build did not reach a healthy terminal state in time

For build failures, classify the cause before suggesting a fix:
- repo/bootstrap failure
- GitHub permission failure
- app build failure
- runtime image or port mismatch

If deployment/build state says `building` but the user believes “no build pod started”, verify the workflow before agreeing:

```bash
bifrost build get <build-id> --json --non-interactive
kubectl -n bifrost get wf <argo-workflow-ref>
kubectl -n bifrost logs <argo-workflow-pod> -c main --tail=200
```

This often separates:
- pod scheduling delay
- private repo clone/auth failure
- real app build failure

If a build succeeds but deployment fails, stop creating more deployments. Move to deployment/runtime diagnosis first.

Framework-specific references can help keep the first fix precise:
- `dockerfile-guidance.md`
- `framework-nextjs.md`
- `framework-vite-static.md`
- `framework-node-service.md`
- `runtime-diagnosis.md`

## Deployment Timeout or Unhealthy Rollout

Use:

```bash
bifrost deployment get <deployment-id> --json --non-interactive
bifrost deployment wait <deployment-id> --json --non-interactive
```

Prefer these commands before reaching for lower-level platform debugging.
If deployment succeeds, always fetch and return the public URL without waiting for the user to ask.

If the route exists but the public URL is still returning `502`, `503`, or `fault filter abort`, classify that as:
- `routing in progress` while edge propagation is converging

Only treat it as `edge ready` after repeated healthy public responses.

If the rollout is healthy but the app needs a controlled bounce:

```bash
bifrost deployment restart <deployment-id> --json --non-interactive
```

If env vars or secrets were updated and the running release needs to pick them up:

```bash
bifrost service apply-config <service> --project <project> --environment <env> --json --non-interactive
```
