# Litmus Deploy Marker Action

Post a deploy marker to [Litmus](https://trylitmus.app) after a successful deploy. The dashboard renders markers as vertical dashed lines on the behavioral-rate charts so a shift in abandon, edit, or regen rate lines up with the exact release that caused it.

Built to sit right next to your Sentry release step — same shape, same ergonomics.

## Usage

```yaml
- uses: trylitmus/deploy-marker-action@v1
  continue-on-error: true
  with:
    api-key: ${{ secrets.LITMUS_DEPLOYS_API_KEY }}
    service: your-service-name
    environment: ${{ github.ref_name }}
```

## Setup

1. In the Litmus dashboard, create a **secret key** (`ltm_sk_live_*`) with the `deploys:write` scope.
2. Add it to your repo as the `LITMUS_DEPLOYS_API_KEY` secret.
3. Drop the action into your deploy workflow **after** the deploy succeeds.

Your browser/backend SDK key is a **publishable** key (`ltm_pk_*`) and cannot create markers. This is intentional — publishable keys are shipped in bundles.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | yes | — | Litmus secret key (`ltm_sk_*`) with `deploys:write` scope |
| `service` | yes | — | Service name, written to `metadata.service` |
| `environment` | no | `production` | Deploy environment |
| `version` | no | `github.sha[:7]` | Version string shown on the chart |
| `description` | no | `<service> <environment> deploy` | Human-readable label |
| `api-url` | no | `https://ingest.trylitmus.app` | Override for self-hosted or staging |

## Full example

```yaml
name: Deploy
on:
  push:
    branches: [production]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # ... your build + deploy steps ...

      - uses: getsentry/action-release@v3
        continue-on-error: true
        env:
          SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
          SENTRY_ORG: ${{ vars.SENTRY_ORG }}
          SENTRY_PROJECT: my-service
        with:
          environment: ${{ github.ref_name }}

      - uses: trylitmus/deploy-marker-action@v1
        continue-on-error: true
        with:
          api-key: ${{ secrets.LITMUS_DEPLOYS_API_KEY }}
          service: my-service
          environment: ${{ github.ref_name }}
```

## Behavior

- Skips quietly (with a warning annotation) when `api-key` is empty — safe to add before the secret is wired up.
- `--connect-timeout 10 --max-time 30 --retry 2` — a flaky Litmus endpoint never blocks your deploy.
- Pair with `continue-on-error: true` if you want marker failures to never fail the job.
- Payload built with `jq --argjson` for `run_number` so it's emitted as a JSON number, not a string.

## Pinning

- `@v1` tracks the latest `v1.x.x` release (recommended)
- `@v1.0.0` pins to an exact release
- `@main` tracks the latest commit (not recommended for prod)

## License

MIT
