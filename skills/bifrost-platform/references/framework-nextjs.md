# Next.js Deploy Expectations

Use this when the repo is clearly a Next.js application.

## Defaults

- containerized Next.js app: project type `monolith`
- static-export Next.js app: project type `static_site`
- likely runtime port for containerized Next.js: `3000`
- likely health path: `/`

## Preflight Checks

- inspect `package.json` for `next build` and `next start`
- inspect `next.config.*`
- determine which `output` mode is configured:
  - `output: 'export'`
  - `output: 'standalone'`
- verify whether the app uses server features such as:
  - server-side data fetching
  - API routes
  - database access during request handling
  - auth/session middleware that requires a running server
- verify whether env vars such as `NEXT_PUBLIC_*` and server-only vars are present

## Output Mode Decision Rules

### `output: 'export'`
Treat this as a static-site candidate when the app is truly static.

Use `static_site` when:
- the app is a static export
- pages can be generated at build time
- there are no request-time server/database requirements

For Bifrost static hosting, exported Next.js sites must be built for the project subpath.
Prefer setting an appropriate `basePath`, for example:
- `basePath: '/s/<project-slug>'`

Without that, CSS and JS assets will be emitted from `/_next/...` and break when served under `/s/<project-slug>/...`.

If the repo is clearly meant to be static but `output: 'export'` is missing or the static path config is wrong, the agent should fix it before deploying.

### `output: 'standalone'`
Treat this as a containerized application by default.

Use `monolith`/container deploy when:
- the app needs a running Next.js server
- the app performs request-time backend or DB work
- the app exposes API routes or other dynamic server behavior

If the user selected `static_site` but the repo is configured with `output: 'standalone'`, the agent should not blindly deploy it as static.
It should:
- inspect whether the app is actually dynamic
- if the app is truly dynamic, deploy it as a containerized application
- if the app is intended to be static, change it to `output: 'export'` and then deploy it as `static_site`

## Common Runtime Requirements

- container should bind `0.0.0.0`
- Bifrost service port should match the actual runtime port
- if the app uses a database, verify connection settings before blaming deployment

## Common Failure Modes

- service configured for `80` while app listens on `3000`
- server binds only to localhost
- DB route failures mistaken for full deployment failure
- successful build followed by routing lag at the edge
- static export published successfully but CSS/JS missing because `basePath` was not configured for `/s/<project-slug>/...`

## Skill Behavior

- after push, check for a webhook deployment for the commit before creating a manual deployment
- if the root page returns `200` but a data route fails, classify that as an app/runtime config issue, not a total deploy failure
- for static export, align the Next.js config with Bifrost static hosting before deploying
- use `output: 'standalone'` plus container deploy for genuinely dynamic Next.js apps
