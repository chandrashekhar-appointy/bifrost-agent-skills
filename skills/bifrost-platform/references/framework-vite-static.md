# Vite / Static Site Deploy Expectations

Use this when the repo builds static assets and serves them with nginx or another static server.

## Defaults

- project type: `static_site` when the app can be published as pure assets
- fallback project type: `monolith` only when the app still requires a long-running container runtime
- preferred service kind: `static_site`
- fallback runtime port: `8080` only when the app must still run in a non-root nginx container
- likely health path: `/`

## Preflight Checks

- inspect `package.json` for `vite build`
- inspect `vite.config.*` for `base`
- check whether the repo serves static output through:
  - nginx
  - a custom static server
  - another HTTP server
- verify whether the generated asset paths are safe for Bifrost static hosting under `/s/<project-slug>/...`

## Static Hosting Requirement

Bifrost static sites are served from a path prefix, not the domain root.

That means Vite apps should use static-safe asset paths:
- prefer `base: './'` for generic path-safe static output
- absolute root paths like `/assets/...` are unsafe for Bifrost static hosting
- internal page links should also avoid hard-coding domain-root paths when the site is meant to live under `/s/<project-slug>/...`

If the repo is otherwise a good static-site candidate but `vite.config.*` is missing a safe `base`, the agent should fix that before deployment.

## Common Failure Modes

- nginx temp/cache permission errors at runtime
- image builds successfully but the container crashloops
- static assets built for the wrong `base`
- service port and container port mismatch
- blank pages caused by JS/CSS bundles being requested from `/assets/...` instead of the project subpath

## Nginx-Specific Rules

- use non-root-safe runtime paths
- prefer `/tmp` for temp/cache directories
- avoid assuming `/var/cache/nginx` is writable
- listen on `8080` instead of privileged ports when running non-root

## Skill Behavior

- prefer the shared-bucket/CDN static hosting path first; see `static-site-hosting.md`
- if the app is going to Bifrost static hosting, make the Vite build subpath-safe before deploying
- after the first successful build + failed rollout, inspect pod logs immediately
- do not keep creating new deployments until the runtime crash cause is identified
