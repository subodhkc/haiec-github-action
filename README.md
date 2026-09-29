# HAIEC GitHub Action v2

GitHub Action v2 runs the HAIEC local scanner inside the customer GitHub runner. It submits a qualified Evidence Bundle to HAIEC and starts the canonical Assurance Run.

## Architecture

```text
actions/checkout
  → @haiec/cli@0.1.0 scan local
  → POST /api/v1/scans
  → POST /api/v1/assurance/runs
  → Evaluation / Report / Passport
```

The Action does not send `GITHUB_TOKEN`, `githubToken`, or `X-GitHub-Token` to HAIEC. `${{ github.token }}` is used only by GitHub-native SARIF upload.

## Usage

```yaml
permissions:
  contents: read
  security-events: write

steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0

  - uses: subodhkc/haiec-github-action@v2
    id: haiec
    with:
      haiec-api-key: ${{ secrets.HAIEC_API_KEY }}
      ai-system-id: ${{ secrets.HAIEC_AI_SYSTEM_ID }}
      wait: 'true'
      upload-sarif: 'true'
```

Required API key scopes: `scan:submit`, `scan:create`, `scan:read`, and `report:read`.

## Outputs

- `scan-id`
- `run-id`
- `evaluation-id`
- `disposition`
- `report-url`
- `passport-url`
- `findings-total`
- `findings-critical`
- `findings-high`

## Migration from v1

v1 is a legacy remote compatibility action and is deprecated. v2 requires `ai-system-id`, runs analysis in the customer runner, and does not accept or transmit a GitHub token to HAIEC.

v2 should be tagged only after `@haiec/cli@0.1.0` is published and the release E2E gates pass.
