# Litmus Deploy Marker Action

Post a deploy marker to [Litmus](https://trylitmus.app) after a successful deploy. The dashboard renders markers as vertical dashed lines on the behavioral-rate charts, so a shift in abandon, edit, or regen rate lines up with the exact release that caused it.

## Usage

```yaml
- uses: trylitmus/deploy-marker-action@v1
  continue-on-error: true
  env:
    LITMUS_AUTH_TOKEN: ${{ secrets.LITMUS_AUTH_TOKEN }}
    LITMUS_SERVICE: my-service
  with:
    environment: ${{ github.ref_name }}
```

## Setup

1. In the Litmus dashboard, create a **secret key** (`ltm_sk_live_*`) with the `deploys:write` scope.
2. Add it to your repo as the `LITMUS_AUTH_TOKEN` secret.
3. Drop the action into your deploy workflow after the deploy step succeeds.

Your SDK key is a **publishable** key (`ltm_pk_*`) and cannot create markers. This is intentional — publishable keys are shipped in bundles.

## Environment variables

| Variable | Required | Description |
|---|---|---|
| `LITMUS_AUTH_TOKEN` | yes | Litmus secret key (`ltm_sk_*`) with `deploys:write` scope |
| `LITMUS_SERVICE` | yes | Service name, written to `metadata.service` so multiple services on one project can be distinguished |
| `LITMUS_URL` | no | Override the ingest URL. Defaults to `https://ingest.trylitmus.app` |

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `environment` | no | `production` | Deploy environment |
| `version` | no | `github.sha[:7]` | Version string shown on the chart |
| `description` | no | `<service> <environment> deploy` | Human-readable label |

## Multi-service example

One repo, two services sharing a Litmus project — tell them apart via `LITMUS_SERVICE`:

```yaml
- uses: trylitmus/deploy-marker-action@v1
  continue-on-error: true
  env:
    LITMUS_AUTH_TOKEN: ${{ secrets.LITMUS_AUTH_TOKEN }}
    LITMUS_SERVICE: api
  with:
    environment: ${{ github.ref_name }}

- uses: trylitmus/deploy-marker-action@v1
  continue-on-error: true
  env:
    LITMUS_AUTH_TOKEN: ${{ secrets.LITMUS_AUTH_TOKEN }}
    LITMUS_SERVICE: worker
  with:
    environment: ${{ github.ref_name }}
```

## Behavior

- Skips quietly (warning annotation, exit 0) when `LITMUS_AUTH_TOKEN` is empty — safe to merge before the secret is wired up.
- `curl --connect-timeout 10 --max-time 30 --retry 2` so a flaky endpoint never blocks your deploy.
- Pair with `continue-on-error: true` if you want marker failures to never fail the job.
- Payload built with `jq --argjson` so `metadata.run_number` is a real JSON number, and commit messages / refs can't inject into the body.

## Pinning

- `@v1` tracks the latest `v1.x.x` release (recommended)
- `@v1.0.0` pins to an exact release
- `@main` tracks the latest commit (not recommended for prod)

## License

MIT
