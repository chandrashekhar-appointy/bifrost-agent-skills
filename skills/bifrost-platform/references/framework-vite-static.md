# Vite / Static Site Deploy Expectations

Use this when the repo builds static assets and serves them with nginx or another static server.

## Defaults

- project type: `monolith`
- likely runtime port: `8080` for non-root nginx containers
- likely health path: `/`

## Preflight Checks

- inspect `package.json` for `vite build`
- inspect `vite.config.*` for `base`
- check whether the repo serves static output through:
  - nginx
  - a custom static server
  - another HTTP server

## Common Failure Modes

- nginx temp/cache permission errors at runtime
- image builds successfully but the container crashloops
- static assets built for the wrong `base`
- service port and container port mismatch

## Nginx-Specific Rules

- use non-root-safe runtime paths
- prefer `/tmp` for temp/cache directories
- avoid assuming `/var/cache/nginx` is writable
- listen on `8080` instead of privileged ports when running non-root

## Skill Behavior

- after the first successful build + failed rollout, inspect pod logs immediately
- do not keep creating new deployments until the runtime crash cause is identified
