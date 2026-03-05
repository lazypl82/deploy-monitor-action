# Relivio Deploy Monitor

Send one deployment event to Relivio from your GitHub Actions deploy pipeline.

This action calls:

- `POST /api/v1/deployments`

## Quick Start

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # your deploy steps...

      - name: Notify Relivio
        uses: lazypl82/deploy-monitor-action@v1
        with:
          api-key: ${{ secrets.RELIVIO_PROJECT_API_KEY }}
          environment: production
          idempotency-key: deploy-${{ github.sha }}
```

## What This Sends

The action sends:

- `version`
- `note` (optional)
- `metadata` (optional):
  - `environment`
  - `repo`
  - `commit`
  - `workflow`
  - `run_id`

When `include-github-context: true` (default), GitHub context fields are attached to `metadata`.

## Inputs

- `api-key` (required): Relivio project runtime API key
- `api-base-url` (optional, default: `https://api.relivio.dev`)
- `version` (optional, default: `GITHUB_SHA`)
- `note` (optional)
- `environment` (optional, default: `production`)
- `include-github-context` (optional, default: `true`)
- `legacy-fallback` (optional, default: `true`): retry once without `metadata` when server returns `422`
- `idempotency-key` (optional)
- `timeout-seconds` (optional, default: `15`)

## Outputs

- `deployment-id`: deployment id returned by Relivio
- `summary-scheduled`: `true` or `false`
- `status-code`: HTTP status code
- `used-legacy-fallback`: `true` when request retried without metadata

## Recommended

- Set `idempotency-key: deploy-${{ github.sha }}` for retry-safe deploy registration.
- Keep `api-key` only in GitHub Secrets.
- Keep `include-github-context: true` to preserve repo/commit/workflow trace in structured metadata.

## Compatibility Note

If your Relivio server does not yet support `metadata` on `POST /api/v1/deployments`, keep `legacy-fallback: true` (default).  
The action will retry once using a legacy payload (`version` + `note`) when the first request returns `422`.
