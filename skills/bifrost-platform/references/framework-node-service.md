# Node Service Deploy Expectations

Use this when the repo is a long-running Node backend or API service.

## Defaults

- project type: `monolith`
- likely runtime port: `3000`, `4000`, or `8080`
- prefer an explicit health endpoint when the app provides one

## Preflight Checks

- inspect `package.json` scripts
- inspect the server entrypoint for the listening host and port
- inspect env expectations such as `PORT`, `HOST`, `DATABASE_URL`, `REDIS_URL`

## Common Failure Modes

- app binds to `localhost` instead of `0.0.0.0`
- wrong port configured in the service
- missing runtime env vars or secrets
- health path configured to a route that requires auth or extra infra

## Skill Behavior

- choose the service port from the actual app runtime, not a generic default
- if env/config changes are the likely cause, prefer `bifrost service apply-config ...` over a full image redeploy
