# Static Site Hosting

Use this flow when the repo builds static assets and does not need a long-running application container.

In the Bifrost product UI, this should be chosen at the **project type** level as `Static Site`.
Do not ask the user to choose `Service Kind` again for the same project in v1.

## Choose `static_site`

Prefer `static_site` when the repo is clearly a frontend/static site:
- Vite or similar client-only build
- Next.js with `output: 'export'`
- output like `dist/`, `build/`, or `out/`
- no server runtime, worker, or API process
- no need for Kubernetes networking, Services, or `HTTPRoute`

Do **not** use `static_site` when the app is actually dynamic at request time, even if it is a frontend framework repo.
For example:
- Next.js with `output: 'standalone'` and API/server behavior
- apps that need server-side DB access during requests
- apps that require a long-running application process

The CLI still represents this as a service kind internally. Create the service with:

```bash
bifrost project create <project> --type static_site --json --non-interactive

bifrost service create <name>   --project <project>   --kind static_site   --repo <repo-url>   --branch <branch>   --root <root-dir>   --install-command "npm ci"   --build-command "npm run build"   --output-dir dist   --json --non-interactive
```

## Subpath Requirement

Bifrost static sites are served from:
- `https://static.bifrost.saastack.site/s/<project-slug>/...`

That means the generated site must be **subpath-safe**.

Examples:
- Vite should use relative/static-safe asset paths such as `base: './'`
- Next.js static export should use an appropriate `basePath`, such as `/s/<project-slug>`

If the repo is otherwise a valid static-site candidate but the generated asset paths are rooted at `/`, the agent should fix that before deployment.

## Next.js Rule of Thumb

- `output: 'export'` + static behavior => deploy as `static_site`
- `output: 'standalone'` + dynamic/server behavior => deploy as containerized app
- `output: 'standalone'` + user intended a static site => change the app to `output: 'export'` before static deployment

## Deploy Behavior

For `static_site` services:
- do not require a user-visible environment
- deploy through `bifrost deploy` without `--environment`
- expect Bifrost to:
  - clone the repo
  - run install/build commands
  - upload assets to the shared static bucket
  - sync the `live/` alias
  - invalidate CDN cache
  - wait for the public URL to return healthy responses

Example:

```bash
bifrost deploy --project <project> --service <service> --wait --json --non-interactive
```

## URL Contract

Stable URL:

```text
https://static.bifrost.saastack.site/s/<project-slug>/
```

Immutable release URL:

```text
https://static.bifrost.saastack.site/s/<project-slug>/r/<release-id>/
```

## Agent Behavior

- ask fewer infra/runtime questions than for container services
- do not create or expect Kubernetes environments for static sites
- do not expect `HTTPRoute`, pod logs, or rollout diagnostics
- return both:
  - stable URL
  - release URL
- if publish succeeds but the URL is not live yet, report `routing in progress` until the edge probes pass
