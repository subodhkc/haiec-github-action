# HAIEC GitHub Action

AI security scanning for your codebase - detect prompt injection, data leakage, and AI-specific vulnerabilities in your CI/CD pipeline.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-HAIEC%20Security%20Scan-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/haiec-security-scan)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Links

- **HAIEC Platform:** [haiec.com](https://haiec.com) - AI governance and compliance
- **Author:** [Subodh KC](https://subodhkc.com) - AI governance, compliance, and security leader

---

## Features

- AI-Specific Security Rules - Detect prompt injection, data leakage, and AI model vulnerabilities
- Fast Scanning - Optimized for CI/CD with incremental analysis
- GitHub Integration - SARIF upload to GitHub Code Scanning
- Policy Gates - Fail builds based on severity thresholds
- Risk Scoring - Quantitative security assessment (0-100)
- Detailed Reports - HTML and JSON output with remediation guidance

## Quick Start

### 1. Get Your API Key

1. Sign up at [haiec.com](https://haiec.com)
2. Navigate to **Dashboard -> API Keys**
3. Click **Generate New Key**
4. Copy your API key (starts with `haiec_live_` or `haiec_test_`)

### 2. Add Secret to GitHub

1. Go to your repository **Settings -> Secrets and variables -> Actions**
2. Click **New repository secret**
3. Name: `HAIEC_API_KEY`
4. Value: Your API key from step 1
5. Click **Add secret**

### 3. Create Workflow

Create `.github/workflows/haiec-security.yml`:

```yaml
name: HAIEC Security Scan

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  security-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
      pull-requests: write
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Run HAIEC Security Scan
        uses: subodhkc/haiec-github-action@v1
        with:
          haiec-api-key: ${{ secrets.HAIEC_API_KEY }}
      
      - name: Upload scan results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: haiec-security-results
          path: |
            haiec-results.json
            haiec-report.html
          retention-days: 30
      
      - name: Upload SARIF to GitHub Security
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: haiec-results/haiec-scan.sarif
```

## Configuration

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `haiec-api-key` | HAIEC API key | Yes | - |
| `fail-on-critical` | Fail build on critical findings | No | `true` |
| `fail-on-high` | Fail build on high findings | No | `false` |
| `upload-sarif` | Upload SARIF to GitHub | No | `true` |
| `policy-file` | Path to custom policy file | No | `''` |
| `scan-paths` | Paths to scan (comma-separated) | No | `''` (all) |
| `exclude-paths` | Paths to exclude | No | `node_modules/**,vendor/**` |

### Outputs

| Output | Description |
|--------|-------------|
| `scan-id` | Unique scan identifier |
| `findings-count` | Total number of findings |
| `critical-count` | Number of critical findings |
| `high-count` | Number of high findings |
| `gate-passed` | Whether policy gate passed |
| `risk-score` | Overall risk score (0-100) |

## Advanced Usage

### Custom Policy Gates

```yaml
- name: Run HAIEC Security Scan
  uses: subodhkc/haiec-github-action@v1
  with:
    haiec-api-key: ${{ secrets.HAIEC_API_KEY }}
    fail-on-critical: 'true'
    fail-on-high: 'true'
    exclude-paths: 'node_modules/**,test/**,*.test.ts'
```

### Scheduled Scans

```yaml
on:
  schedule:
    - cron: '0 2 * * 1'
  workflow_dispatch:
```

## Output Files

- **`haiec-results.json`** - Complete scan results in JSON format
- **`haiec-report.html`** - Interactive HTML report
- **`haiec-results/haiec-scan.sarif`** - SARIF format for GitHub Code Scanning

## Supported Languages

- JavaScript/TypeScript (Node.js, React, Vue, Angular)
- Python (Django, Flask, FastAPI)
- Go
- Java/Kotlin
- Ruby
- PHP

## Security Rules

HAIEC detects AI-specific vulnerabilities including:

- Prompt Injection - User input flowing to LLM prompts
- Data Leakage - Sensitive data in prompts or responses
- Model Poisoning - Unsafe model loading or training
- API Key Exposure - Hardcoded API keys and secrets
- Insecure Deserialization - Unsafe pickle/YAML loading
- Path Traversal - File system access vulnerabilities
- SQL Injection - Database query vulnerabilities
- XSS - Cross-site scripting in AI outputs

[View all security rules](https://haiec.com/docs/rules)

## Troubleshooting

### 401 Unauthorized
Verify `HAIEC_API_KEY` secret exists in repository settings. Check API key is active at https://haiec.com/dashboard/api-keys

### No Artifacts Uploaded
Check action logs for errors. Verify API key authentication. Ensure `if: always()` is set on upload steps.

### Permission Denied: security-events
Enable GitHub Advanced Security in repository settings. Or remove `security-events: write` permission if not needed.

### Rate Limit Exceeded
Check usage at https://haiec.com/dashboard/usage. Upgrade plan at https://haiec.com/pricing.

## Support

- [Documentation](https://haiec.com/docs)
- [Discord Community](https://discord.gg/haiec)
- [Email Support](mailto:support@haiec.com)
- [Report Issues](https://github.com/subodhkc/haiec-github-action/issues)

## License

MIT License - see [LICENSE](LICENSE) for details

## Links

- [HAIEC Platform](https://haiec.com)
- [Dashboard](https://haiec.com/dashboard)
- [API Documentation](https://haiec.com/docs/api)
- [Security Rules](https://haiec.com/docs/rules)
- [Pricing](https://haiec.com/pricing)
- [Subodh KC](https://subodhkc.com)