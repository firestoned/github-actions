# Trivy Container Security Scan

A composite GitHub Action that scans container images for vulnerabilities using Trivy, with automatic integration into GitHub Security tab and detailed JSON reporting.

## Features

- Comprehensive container image vulnerability scanning
- Automatic upload to GitHub Security tab (SARIF format)
- Detailed JSON reports for offline analysis
- Multi-severity scanning (CRITICAL, HIGH, MEDIUM, LOW)
- Supports all major container registries (GHCR, Docker Hub, etc.)
- Scans OS packages, language dependencies, and libraries
- Always-upload pattern (reports generated even on failure)
- Customizable output filenames and artifact names

## Usage

### Basic Example

```yaml
- name: Scan container image
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: ghcr.io/${{ github.repository }}:latest
```

### With Custom Artifact Name

```yaml
- name: Scan container image
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: ghcr.io/${{ github.repository }}:${{ github.sha }}
    upload-artifact-name: trivy-scan-${{ github.run_id }}
```

### Custom SARIF Category

```yaml
- name: Scan container image
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: myregistry.io/myapp:v1.0.0
    sarif-category: trivy-production-scan
```

### Complete Workflow

```yaml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t myapp:test .

      - name: Scan with Trivy
        uses: your-org/github-actions/security/trivy-scan@v1
        with:
          image-ref: myapp:test
          sarif-category: trivy-pr-scan

      - name: Download scan results
        uses: actions/download-artifact@v4
        with:
          name: trivy-scan-report
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `image-ref` | Container image reference to scan (e.g., `ghcr.io/org/repo:tag`) | Yes | N/A |
| `sarif-category` | Category name for SARIF upload to GitHub Security tab | No | `trivy-container-scan` |
| `upload-artifact-name` | Name for the Trivy scan report artifact | No | `trivy-scan-report` |
| `output-filename` | Filename for the JSON output report | No | `trivy-results.json` |

## Outputs

### SARIF Report

Automatically uploaded to GitHub Security tab:
- View at: `https://github.com/{owner}/{repo}/security/code-scanning`
- Severity filtering: CRITICAL and HIGH only
- Integrated with GitHub's security features
- Triggers security advisories

### JSON Report

Uploaded as workflow artifact:
- Filename: Specified in `output-filename` (default: `trivy-results.json`)
- Artifact name: Specified in `upload-artifact-name`
- Severity: ALL levels (CRITICAL, HIGH, MEDIUM, LOW)
- Detailed vulnerability information

## How It Works

### Scan Process

1. **SARIF Scan**: Scans for CRITICAL and HIGH vulnerabilities
   - Uses `continue-on-error: true` to always proceed
   - Generates `trivy-results.sarif`

2. **Upload to Security Tab**: Uploads SARIF to GitHub
   - Visible in Security > Code scanning alerts
   - Categorized by `sarif-category`

3. **JSON Scan**: Comprehensive scan for all severities
   - Generates detailed JSON report
   - Includes CRITICAL, HIGH, MEDIUM, LOW

4. **Upload Artifact**: Archives JSON report
   - Available in workflow artifacts
   - Persists for offline analysis

### Scan Coverage

Trivy scans for vulnerabilities in:
- **OS packages**: apt, yum, apk, etc.
- **Language dependencies**: npm, pip, gem, cargo, go.mod, etc.
- **Application libraries**: JAR files, Python wheels, etc.
- **Container layers**: All filesystem layers

## Required Permissions

For GitHub Security tab integration:

```yaml
permissions:
  security-events: write  # Required for SARIF upload
  contents: read          # Required for checkout
```

## Best Practices

### 1. Scan After Building

Always scan immediately after building:

```yaml
- name: Build image
  uses: docker/build-push-action@v5
  with:
    push: false
    load: true
    tags: myapp:${{ github.sha }}

- name: Scan image
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: myapp:${{ github.sha }}
```

### 2. Scan Before Pushing

Gate pushes on security scans:

```yaml
- name: Scan image
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: myapp:${{ github.sha }}

- name: Push image (only if scan passes)
  if: success()
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: ghcr.io/${{ github.repository }}:latest
```

### 3. Use Image Digests

Scan specific image digests for reproducibility:

```yaml
- name: Build and capture digest
  id: build
  uses: docker/build-push-action@v5
  with:
    outputs: type=docker

- name: Scan by digest
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: ${{ steps.build.outputs.digest }}
```

### 4. Categorize Scans

Use different categories for different environments:

```yaml
# For PR scans
sarif-category: trivy-pr-scan

# For production scans
sarif-category: trivy-production-scan

# For staging scans
sarif-category: trivy-staging-scan
```

### 5. Archive Scan Results

Keep scan results for compliance:

```yaml
- name: Scan and archive
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: myapp:${{ github.sha }}
    upload-artifact-name: trivy-scan-${{ github.sha }}
    output-filename: trivy-${{ github.sha }}.json
```

## Troubleshooting

### SARIF Upload Fails

**Problem**: "Resource not accessible by integration"

**Solution**: Add required permissions:
```yaml
permissions:
  security-events: write
  contents: read
```

### Image Not Found

**Problem**: "Unable to find image 'myapp:tag'"

**Solution**: Ensure image is loaded in Docker daemon:
```yaml
- uses: docker/build-push-action@v5
  with:
    load: true  # Important!
    tags: myapp:tag
```

### No Vulnerabilities in Security Tab

**Problem**: Scan succeeds but no alerts in Security tab

**Solution**: Check severity filter in Security tab settings. The SARIF scan only includes CRITICAL and HIGH.

### Scan Times Out

**Problem**: Large images cause timeout

**Solution**: Increase timeout or use Trivy caching:
```yaml
- name: Cache Trivy DB
  uses: actions/cache@v4
  with:
    path: ~/.cache/trivy
    key: trivy-db-${{ github.run_id }}
    restore-keys: trivy-db-
```

## Advanced Usage

### Custom Severity Threshold

To fail on specific severities, check the JSON report:

```yaml
- name: Scan image
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: myapp:test

- name: Check for CRITICAL vulnerabilities
  run: |
    CRITICAL=$(jq '[.Results[].Vulnerabilities[]? | select(.Severity=="CRITICAL")] | length' trivy-results.json)
    if [ "$CRITICAL" -gt 0 ]; then
      echo "Found $CRITICAL CRITICAL vulnerabilities"
      exit 1
    fi
```

### Scan Multiple Images

Use matrix builds:

```yaml
strategy:
  matrix:
    image:
      - myapp:latest
      - myapp:alpine
      - myapp:distroless
steps:
  - uses: your-org/github-actions/security/trivy-scan@v1
    with:
      image-ref: ${{ matrix.image }}
      sarif-category: trivy-${{ matrix.image }}
```

### Ignore Specific Vulnerabilities

Create a `.trivyignore` file:

```
# .trivyignore
CVE-2023-12345
CVE-2023-67890
```

### Scan SBOM Instead of Image

Scan a CycloneDX SBOM:

```yaml
- name: Scan SBOM
  uses: aquasecurity/trivy-action@master
  with:
    scan-type: 'sbom'
    input: my-app.cdx.json
    format: 'sarif'
    output: 'trivy-sbom.sarif'
```

## Understanding Scan Results

### SARIF Format (GitHub Security Tab)

```json
{
  "runs": [{
    "results": [{
      "ruleId": "CVE-2023-12345",
      "level": "error",
      "message": {
        "text": "Package foo version 1.2.3 is vulnerable to CVE-2023-12345"
      },
      "locations": [{
        "physicalLocation": {
          "artifactLocation": {
            "uri": "usr/bin/foo"
          }
        }
      }]
    }]
  }]
}
```

### JSON Format (Artifact)

```json
{
  "Results": [
    {
      "Target": "alpine:3.18 (alpine 3.18.0)",
      "Type": "alpine",
      "Vulnerabilities": [
        {
          "VulnerabilityID": "CVE-2023-12345",
          "PkgName": "foo",
          "InstalledVersion": "1.2.3-r0",
          "FixedVersion": "1.2.4-r0",
          "Severity": "HIGH",
          "Title": "Buffer overflow in foo",
          "Description": "...",
          "PrimaryURL": "https://avd.aquasec.com/nvd/cve-2023-12345"
        }
      ]
    }
  ]
}
```

## Scan Strategies

### 1. Gating Strategy

Block deployments on vulnerabilities:

```yaml
- name: Scan
  id: scan
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: ${{ env.IMAGE }}

- name: Check results
  run: |
    if jq -e '.Results[].Vulnerabilities[]? | select(.Severity=="CRITICAL")' trivy-results.json; then
      echo "CRITICAL vulnerabilities found - blocking deployment"
      exit 1
    fi

- name: Deploy (only if no CRITICAL)
  run: kubectl apply -f deployment.yaml
```

### 2. Monitoring Strategy

Scan regularly but don't block:

```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Daily

- uses: your-org/github-actions/security/trivy-scan@v1
  continue-on-error: true
  with:
    image-ref: ghcr.io/${{ github.repository }}:latest
```

### 3. Progressive Strategy

Different thresholds for different environments:

```yaml
# Development: Warn only
- uses: your-org/github-actions/security/trivy-scan@v1
  if: github.ref != 'refs/heads/main'
  continue-on-error: true

# Production: Block on HIGH+
- uses: your-org/github-actions/security/trivy-scan@v1
  if: github.ref == 'refs/heads/main'
```

## Performance Tips

### 1. Cache Trivy Database

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/trivy
    key: trivy-db-${{ hashFiles('**/Dockerfile') }}
```

### 2. Scan in Parallel

```yaml
- name: Build images
  run: |
    docker build -t app:latest .
    docker build -t app:alpine -f Dockerfile.alpine .

- name: Scan latest
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: app:latest

- name: Scan alpine (parallel)
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: app:alpine
```

### 3. Skip Unnecessary Scans

```yaml
- name: Check if Dockerfile changed
  id: check
  run: echo "changed=$(git diff --name-only HEAD^ | grep Dockerfile)" >> $GITHUB_OUTPUT

- name: Scan (only if Dockerfile changed)
  if: steps.check.outputs.changed
  uses: your-org/github-actions/security/trivy-scan@v1
```

## Related Actions

- [rust/security-scan](../../rust/security-scan/README.md) - Scan Rust dependencies
- [security/cosign-sign](../cosign-sign/README.md) - Sign container images
- [docker/setup-docker](../../docker/setup-docker/README.md) - Setup Docker environment
- [rust/generate-sbom](../../rust/generate-sbom/README.md) - Generate SBOM for scanning

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Requires Docker
- **Trivy versions**: Uses latest from aquasecurity/trivy-action@master
- **Image formats**: OCI, Docker, containerd

## Security Resources

- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [CVE Database](https://www.cve.org/)
- [GitHub Security Features](https://docs.github.com/en/code-security)
- [SARIF Format](https://sarifweb.azurewebsites.net/)

## Examples

### Complete Security Pipeline

```yaml
name: Security Pipeline

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'

permissions:
  security-events: write
  contents: read
  packages: write

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          load: true
          tags: myapp:${{ github.sha }}

      - name: Scan dependencies
        uses: your-org/github-actions/rust/security-scan@v1

      - name: Scan container
        uses: your-org/github-actions/security/trivy-scan@v1
        with:
          image-ref: myapp:${{ github.sha }}
          sarif-category: trivy-${{ github.event_name }}

      - name: Sign image
        if: github.ref == 'refs/heads/main'
        uses: your-org/github-actions/security/cosign-sign@v1
        with:
          image-digest: ${{ steps.build.outputs.digest }}

      - name: Push to registry
        if: github.ref == 'refs/heads/main'
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```
