# HAIEC GitHub Action

AI security scanning for your codebase - detect prompt injection, data leakage, and AI-specific vulnerabilities in your CI/CD pipeline.

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-HAIEC%20Security%20Scan-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/haiec-security-scan)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- 🛡️ **AI-Specific Security Rules** - Detect prompt injection, data leakage, and AI model vulnerabilities
- 🚀 **Fast Scanning** - Optimized for CI/CD with incremental analysis
- 📊 **GitHub Integration** - SARIF upload to GitHub Code Scanning
- 🎯 **Policy Gates** - Fail builds based on severity thresholds
- 📈 **Risk Scoring** - Quantitative security assessment (0-100)
- 🔍 **Detailed Reports** - HTML and JSON output with remediation guidance

## Quick Start

### 1. Get Your API Key

1. Sign up at [haiec.com](https://haiec.com)
2. Navigate to **Dashboard → API Keys**
3. Click **Generate New Key**
4. Copy your API key (starts with `haiec_live_` or `haiec_test_`)

### 2. Add Secret to GitHub

1. Go to your repository **Settings → Secrets and variables → Actions**
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
| `haiec-api-key` | HAIEC API key | ✅ Yes | - |
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
    - cron: '0 2 * * 1'  # Weekly on Mondays at 2 AM UTC
  workflow_dispatch:
```

### Manual Trigger with Inputs

```yaml
on:
  workflow_dispatch:
    inputs:
      scan_type:
        description: 'Scan type'
        required: false
        default: 'full'
        type: choice
        options:
          - full
          - quick
```

### Severity Threshold Checks

```yaml
- name: Check Critical Findings
  if: always()
  shell: bash
  run: |
    CRITICAL=$(jq -r '.metadata.criticalCount' haiec-results.json)
    if [ "$CRITICAL" -gt 0 ]; then
      echo "::error::Found $CRITICAL critical findings"
      exit 1
    fi
```

### Post Results to PR Comment

```yaml
- name: Comment PR
  if: github.event_name == 'pull_request'
  uses: actions/github-script@v7
  with:
    script: |
      const fs = require('fs');
      const results = JSON.parse(fs.readFileSync('haiec-results.json', 'utf8'));
      const body = `## 🛡️ HAIEC Security Scan
      
      **Status:** ${results.gateResult.pass ? '✅ Passed' : '❌ Failed'}
      
      | Severity | Count |
      |----------|-------|
      | Critical | ${results.metadata.criticalCount} |
      | High | ${results.metadata.highCount} |
      | Medium | ${results.metadata.mediumCount} |
      | Low | ${results.metadata.lowCount} |
      
      [View detailed report](https://haiec.com/dashboard/scans/${results.scanId})`;
      
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: body
      });
```

## Output Files

The action generates the following files:

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
- And more...

## Security Rules

HAIEC detects AI-specific vulnerabilities including:

- **Prompt Injection** - User input flowing to LLM prompts
- **Data Leakage** - Sensitive data in prompts or responses
- **Model Poisoning** - Unsafe model loading or training
- **API Key Exposure** - Hardcoded API keys and secrets
- **Insecure Deserialization** - Unsafe pickle/YAML loading
- **Path Traversal** - File system access vulnerabilities
- **SQL Injection** - Database query vulnerabilities
- **XSS** - Cross-site scripting in AI outputs

[View all security rules →](https://haiec.com/docs/rules)

## Troubleshooting

### 401 Unauthorized

**Cause:** API key not configured or invalid

**Fix:**
1. Verify `HAIEC_API_KEY` secret exists in repository settings
2. Check API key is active at https://haiec.com/dashboard/api-keys
3. Ensure key starts with `haiec_live_` or `haiec_test_`
4. Regenerate key if expired

### No Artifacts Uploaded

**Cause:** Scan failed before generating output files

**Fix:**
1. Check action logs for errors
2. Verify API key authentication
3. Ensure `if: always()` is set on upload steps

### Permission Denied: security-events

**Cause:** GitHub Advanced Security not enabled

**Fix:**
1. Enable GitHub Advanced Security in repository settings
2. Or remove `security-events: write` permission if not needed
3. Public repositories have Advanced Security free

### Rate Limit Exceeded

**Cause:** Exceeded scan quota for your subscription tier

**Fix:**
1. Check usage at https://haiec.com/dashboard/usage
2. Upgrade plan at https://haiec.com/pricing
3. Wait for monthly reset

## Support

- 📚 [Documentation](https://haiec.com/docs)
- 💬 [Discord Community](https://discord.gg/haiec)
- 📧 [Email Support](mailto:support@haiec.com)
- 🐛 [Report Issues](https://github.com/subodhkc/haiec-github-action/issues)

## License

MIT License - see [LICENSE](LICENSE) for details

## Links

- [HAIEC Platform](https://haiec.com)
- [Dashboard](https://haiec.com/dashboard)
- [API Documentation](https://haiec.com/docs/api)
- [Security Rules](https://haiec.com/docs/rules)
- [Pricing](https://haiec.com/pricing)
