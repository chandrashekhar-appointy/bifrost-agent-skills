# Next.js Deploy Expectations

Use this when the repo is clearly a Next.js application.

## Defaults

- project type: `monolith`
- likely runtime port: `3000`
- likely health path: `/`

## Preflight Checks

- inspect `package.json` for `next build` and `next start`
- inspect `next.config.*`
- verify whether the app uses standalone output or a standard server
- verify whether env vars such as `NEXT_PUBLIC_*` and server-only vars are present

## Common Runtime Requirements

- container should bind `0.0.0.0`
- Bifrost service port should match the actual runtime port
- if the app uses a database, verify connection settings before blaming deployment

## Common Failure Modes

- service configured for `80` while app listens on `3000`
- server binds only to localhost
- DB route failures mistaken for full deployment failure
- successful build followed by routing lag at the edge

## Skill Behavior

- after push, check for a webhook deployment for the commit before creating a manual deployment
- if the root page returns `200` but a data route fails, classify that as an app/runtime config issue, not a total deploy failure
