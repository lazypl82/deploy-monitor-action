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
```

## Inputs

- `api-key` (required): Relivio project runtime API key
- `api-base-url` (optional, default: `https://api.relivio.dev`)
- `version` (optional, default: `GITHUB_SHA`)
- `note` (optional)
- `idempotency-key` (optional)

## Outputs

- `deployment-id`: deployment id returned by Relivio
- `status-code`: HTTP status code

## Recommended

- Set `idempotency-key: deploy-${{ github.sha }}` for retry-safe deploy registration.
- Keep `api-key` only in GitHub Secrets.
