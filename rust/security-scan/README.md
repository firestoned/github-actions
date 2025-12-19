# Security Vulnerability Scan

A composite GitHub Action that scans Rust dependencies for known security vulnerabilities using `cargo-audit`, with automatic binary caching and detailed reporting.

## Features

- Scans all dependencies using the RustSec Advisory Database
- Fails builds on any vulnerabilities (CRITICAL, HIGH, MEDIUM, LOW)
- Generates JSON reports for analysis and archiving
- Caches `cargo-audit` binary for fast subsequent runs
- Uploads detailed audit reports as workflow artifacts
- Integrates with cargo dependency caching
- Configurable audit tool version

## Usage

### Basic Example

```yaml
- name: Security scan
  uses: your-org/github-actions/rust/security-scan@v1
```

### Custom cargo-audit Version

```yaml
- name: Security scan
  uses: your-org/github-actions/rust/security-scan@v1
  with:
    cargo-audit-version: '0.21.0'
```

### Custom Artifact Name

```yaml
- name: Security scan
  uses: your-org/github-actions/rust/security-scan@v1
  with:
    upload-artifact-name: 'security-audit-${{ github.run_id }}'
```

### In a Complete CI Workflow

```yaml
name: CI

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Security vulnerability scan
        uses: your-org/github-actions/rust/security-scan@v1

      - name: Download audit report
        if: always()
        uses: actions/download-artifact@v4
        with:
          name: cargo-audit-report
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `cargo-audit-version` | Version of `cargo-audit` to install (e.g., `0.21.0`) | No | `0.21.0` |
| `upload-artifact-name` | Name for the audit report artifact | No | `cargo-audit-report` |

## Outputs

The action uploads a JSON audit report as an artifact with the name specified in `upload-artifact-name`.

### Artifact Contents

The `audit-report.json` file contains:
- Database version and advisory count
- List of all vulnerabilities found
- Vulnerability details (severity, CVEID, description)
- Affected versions and patched versions
- Dependency tree showing how vulnerabilities are introduced

## How It Works

### Scan Process

1. **Install Rust toolchain**: Ensures `cargo` is available
2. **Cache dependencies**: Speeds up dependency resolution
3. **Check binary cache**: Looks for cached `cargo-audit` binary
4. **Install cargo-audit**: Only if not cached
5. **Generate JSON report**: Creates detailed report for archiving
6. **Run security scan**: Fails on any vulnerabilities
7. **Upload artifact**: Always uploads report, even on failure

### Failure Criteria

The action fails if ANY of the following are found:
- CRITICAL vulnerabilities
- HIGH vulnerabilities
- MEDIUM vulnerabilities
- LOW vulnerabilities
- Unmaintained dependencies (with warnings)

### Caching Strategy

Two levels of caching:
- **cargo-audit binary**: Cached by version (saves ~30-60s per run)
- **Cargo dependencies**: Cached by `Cargo.lock` hash

## Best Practices

### 1. Run on Every Push and PR

```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
```

### 2. Schedule Regular Scans

Check for new vulnerabilities daily:

```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
```

### 3. Separate Security Job

Keep security checks separate from builds:

```yaml
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/github-actions/rust/security-scan@v1

  build:
    needs: security
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # ... build steps
```

### 4. Review Reports

Always review the uploaded audit reports:

```yaml
- name: Download audit report
  if: always()
  uses: actions/download-artifact@v4
  with:
    name: cargo-audit-report
```

## Troubleshooting

### Build Fails with Vulnerabilities

**Problem**: Security scan finds vulnerabilities

**Solution**: Review the audit report and take action:

1. **Check the JSON report** in workflow artifacts
2. **Identify vulnerable dependencies**:
   ```bash
   # Locally run audit
   cargo audit
   ```
3. **Update dependencies**:
   ```bash
   cargo update
   cargo audit
   ```
4. **If no patch available**:
   - Check for alternative crates
   - Consider accepting the risk (document in SECURITY.md)
   - Use `--ignore` flag (not recommended)

### cargo-audit Installation Fails

**Problem**: `cargo install cargo-audit` times out or fails

**Solution**:
1. Check network connectivity
2. Verify the version exists: https://crates.io/crates/cargo-audit
3. Try a different version:
   ```yaml
   with:
     cargo-audit-version: '0.20.0'
   ```

### No Audit Report Uploaded

**Problem**: Artifact not found after workflow

**Solution**: The action uses `if: always()` to upload even on failure. Check:
1. Workflow logs for upload step
2. GitHub Actions retention policy (artifacts expire after 90 days by default)
3. Repository artifact storage limits

### False Positives

**Problem**: Audit reports vulnerabilities in unused features

**Solution**:
1. Review dependency tree:
   ```bash
   cargo tree -i <vulnerable-crate>
   ```
2. Disable unused features in `Cargo.toml`:
   ```toml
   [dependencies]
   my-crate = { version = "1.0", default-features = false, features = ["minimal"] }
   ```

## Advanced Usage

### Ignore Specific Advisories

While not recommended, you can ignore specific advisories:

```yaml
- name: Security scan (with ignores)
  shell: bash
  run: |
    cargo audit --deny warnings --ignore RUSTSEC-2023-0001
```

**Important**: Document all ignored advisories in `SECURITY.md` with justification.

### Custom Severity Threshold

To only fail on CRITICAL and HIGH:

```yaml
- name: Security scan (high severity only)
  shell: bash
  run: |
    cargo audit --deny warnings --severity-threshold high
```

### Integrate with Security Tab

Upload results to GitHub Security tab using SARIF:

```yaml
- name: Generate SARIF report
  if: always()
  run: |
    cargo audit --json | cargo-audit-sarif > audit.sarif

- name: Upload to Security tab
  if: always()
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: audit.sarif
```

## Understanding the Audit Report

### JSON Structure

```json
{
  "database": {
    "advisory-count": 500,
    "last-commit": "abc123",
    "last-updated": "2025-01-15"
  },
  "vulnerabilities": {
    "found": true,
    "count": 2,
    "list": [
      {
        "advisory": {
          "id": "RUSTSEC-2023-0001",
          "package": "vulnerable-crate",
          "title": "Buffer overflow in parse function",
          "description": "...",
          "date": "2023-01-15",
          "cvss": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
          "severity": "CRITICAL"
        },
        "versions": {
          "patched": [">=1.5.0"],
          "unaffected": []
        }
      }
    ]
  }
}
```

### Key Fields

- **advisory.id**: RustSec advisory identifier
- **advisory.severity**: CRITICAL, HIGH, MEDIUM, or LOW
- **advisory.cvss**: CVSS v3.1 vector string
- **versions.patched**: Versions that fix the vulnerability
- **package**: Affected crate name

## Performance Tips

### 1. Enable Binary Caching

Already enabled by default. Ensure workflows don't clear caches unnecessarily.

### 2. Use Cargo.lock

Commit `Cargo.lock` to ensure consistent dependency versions:
```bash
git add Cargo.lock
git commit -m "Lock dependencies"
```

### 3. Update Dependencies Regularly

Keep dependencies up-to-date to minimize vulnerabilities:
```bash
cargo update
cargo audit
```

## Related Actions

- [rust/setup-rust-build](../setup-rust-build/README.md) - Setup Rust environment
- [rust/build-binary](../build-binary/README.md) - Build Rust binaries
- [rust/generate-sbom](../generate-sbom/README.md) - Generate Software Bill of Materials
- [security/trivy-scan](../../security/trivy-scan/README.md) - Container image scanning

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Requires internet access to RustSec database
- **Rust versions**: Compatible with all Rust toolchain versions
- **cargo-audit versions**: Tested with 0.20.0 and later

## Security Policy

### Vulnerability Disclosure

If you find a security vulnerability in this action:
1. **Do NOT** open a public issue
2. Email security@firestoned.io with details
3. Allow 90 days for a fix before public disclosure

### RustSec Database

This action uses the [RustSec Advisory Database](https://rustsec.org/):
- Maintained by the Rust security community
- Updated daily with new vulnerabilities
- Open source: https://github.com/RustSec/advisory-db

## Examples

### Complete Security Workflow

```yaml
name: Security

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Rust security scan
        uses: your-org/github-actions/rust/security-scan@v1
        with:
          cargo-audit-version: '0.21.0'
          upload-artifact-name: 'security-audit-${{ github.run_id }}'

      - name: Notify on failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Security vulnerabilities found in ${{ github.repository }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### With Dependency Updates

```yaml
jobs:
  update-and-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Update dependencies
        run: cargo update

      - name: Security scan
        uses: your-org/github-actions/rust/security-scan@v1

      - name: Create PR if vulnerabilities fixed
        if: success()
        uses: peter-evans/create-pull-request@v5
        with:
          commit-message: "chore: update dependencies to fix vulnerabilities"
          title: "Security: Update dependencies"
```
