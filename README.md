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
```

`idempotency-key` defaults to `deploy-<version>` (where `version` falls back to `GITHUB_SHA`), so re-running the same commit deduplicates to the same deployment window. Override only if you intentionally want a different key.

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
- `idempotency-key` (optional, default: `deploy-<version>`)
- `timeout-seconds` (optional, default: `15`)

## Outputs

- `deployment-id`: deployment id returned by Relivio
- `summary-scheduled`: `true` or `false`
- `status-code`: HTTP status code
- `used-legacy-fallback`: `true` when request retried without metadata

## Recommended

- Keep `api-key` only in GitHub Secrets.
- Keep `include-github-context: true` to preserve repo/commit/workflow trace in structured metadata.
- Override `idempotency-key` only when you intentionally want a different deduplication key.

## Other CI Systems (GitLab CI / Jenkins / shell)

This repository ships a GitHub Action. For other CI systems, call the same endpoint with `curl`. The contract is identical.

### GitLab CI

```yaml
notify-relivio:
  stage: deploy
  image: curlimages/curl:8
  script:
    - |
      curl -fsS -X POST "https://api.relivio.dev/api/v1/deployments" \
        -H "Content-Type: application/json" \
        -H "X-API-Key: ${RELIVIO_PROJECT_API_KEY}" \
        -H "Idempotency-Key: deploy-${CI_COMMIT_SHA}" \
        --data "$(printf '{"version":"%s","note":"%s","metadata":{"environment":"production","repo":"%s","commit":"%s","run_id":"%s"}}' \
          "$CI_COMMIT_SHA" "$CI_COMMIT_TITLE" "$CI_PROJECT_PATH" "$CI_COMMIT_SHA" "$CI_PIPELINE_ID")"
```

### Jenkins (declarative pipeline)

```groovy
stage('Notify Relivio') {
  steps {
    withCredentials([string(credentialsId: 'RELIVIO_PROJECT_API_KEY', variable: 'RELIVIO_API_KEY')]) {
      sh '''
        curl -fsS -X POST "https://api.relivio.dev/api/v1/deployments" \
          -H "Content-Type: application/json" \
          -H "X-API-Key: ${RELIVIO_API_KEY}" \
          -H "Idempotency-Key: deploy-${GIT_COMMIT}" \
          --data "$(printf '{"version":"%s","note":"%s","metadata":{"environment":"production","repo":"%s","commit":"%s","run_id":"%s"}}' \
            "$GIT_COMMIT" "$BUILD_TAG" "$JOB_NAME" "$GIT_COMMIT" "$BUILD_ID")"
      '''
    }
  }
}
```

### Generic shell

```bash
curl -fsS -X POST "https://api.relivio.dev/api/v1/deployments" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: ${RELIVIO_API_KEY}" \
  -H "Idempotency-Key: deploy-${COMMIT_SHA}" \
  --data "{\"version\":\"${COMMIT_SHA}\",\"note\":\"${COMMIT_MSG}\"}"
```

### Notes

- Send the `Idempotency-Key` header so retries deduplicate to the same deployment window.
- Keep the API key in your CI secret store; never echo it in logs.
- If your Relivio server does not yet accept `metadata`, drop the `metadata` field and resend (the GitHub Action does this automatically; other clients should handle `422` similarly).

## Compatibility Note

If your Relivio server does not yet support `metadata` on `POST /api/v1/deployments`, keep `legacy-fallback: true` (default).  
The action will retry once using a legacy payload (`version` + `note`) when the first request returns `422`.
