# Image And Template Deploy

Use this reference when the app is deployed from a prebuilt image instead of a GitHub build.

## Two Variants

### Direct image-backed service

Use when the user already has, or wants to build, a container image and deploy it directly.

- create a normal Bifrost project and container service
- set `source_type=image`
- generate upload instructions with:

```bash
bifrost service image-upload <service> --project <project> --image-name <image-name> --image-tag <tag> --json --non-interactive
```

- Bifrost should return a managed Artifact Registry URI under `platform-shared`
- push to the exact URI Bifrost generated
- then deploy normally

Example:

```bash
bifrost service create api \
  --project my-app \
  --source-type image \
  --image-uri us-central1-docker.pkg.dev/<gcp-project>/platform-shared/<prefix>-api:v1 \
  --image-repository us-central1-docker.pkg.dev/<gcp-project>/platform-shared/<prefix>-api \
  --image-tag v1 \
  --json --non-interactive

bifrost deploy --project my-app --service api --environment dev --wait --json --non-interactive
```

### Template launch

Templates are just a higher-level abstraction over:
- project creation
- service creation
- optional infra creation/bind
- image-backed deploy

Use:

```bash
bifrost template list --json --non-interactive
bifrost template get <template-id> --json --non-interactive
bifrost template launch <template-id> --project-name <project> --json --non-interactive
```

Template launch can also take runtime envs and secrets:

```bash
bifrost template launch nginx \
  --project-name my-nginx \
  --env-data 'APP_NAME=my-nginx' \
  --secret-data 'SESSION_SECRET=secret' \
  --json --non-interactive
```

## Upload Command Expectations

For Bifrost-managed image deploys, the generated command should be a buildx push command similar to:

```bash
gcloud auth configure-docker us-central1-docker.pkg.dev
docker buildx build --platform linux/amd64 -t us-central1-docker.pkg.dev/<gcp-project>/platform-shared/<prefix>-<image-name>:<tag> -f Dockerfile --push .
```

Important:
- use the exact managed URI Bifrost returned
- do not assume an older push path is still valid
- image validation checks the exact stored service image URI

## Runtime Config Changes

For image-backed services, runtime envs and secrets are still ordinary service config.

Update them with:

```bash
bifrost service set-env <service> --project <project> --data '{"KEY":"value"}' --json --non-interactive
bifrost service set-secrets <service> --project <project> --data '{"SECRET":"value"}' --json --non-interactive
```

Then apply them to the active release:

```bash
bifrost service apply-config <service> --project <project> --environment <env> --json --non-interactive
```

Use `deployment restart` only when the config is already applied and the workload just needs a bounce.

## Infra Binding

Image-backed services and templates can still use Bifrost-managed infra:
- database
- bucket
- redis
- pub/sub

If infra is created and bound, the bound env vars should be merged into the service config just like repo-backed apps.

## Diagnose Carefully

If deploy says image not found:
- compare the service's stored `image_uri` with the exact Artifact Registry image URI/tag
- confirm the image exists in `platform-shared`
- remember Artifact Registry lists digest URIs while Bifrost may store tag URIs; the platform should resolve both correctly, but mismatched package/tag paths still fail
