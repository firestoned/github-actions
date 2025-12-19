# Sign with Cosign

A composite GitHub Action that signs container images and artifacts using Cosign with keyless signing via Sigstore, supporting both images and files with automatic verification.

## Features

- Keyless signing using Sigstore (no key management needed)
- Sign container images by digest or tag
- Sign arbitrary artifacts (binaries, SBOMs, etc.)
- Automatic signature verification (smoke test)
- Supports multiple image tags
- Generates signature bundles for artifacts
- OIDC-based identity verification
- Transparent signature storage via Rekor

## Usage

### Sign Container Image by Digest

```yaml
- name: Build and push image
  id: build
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: ghcr.io/${{ github.repository }}:latest

- name: Sign image
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    image-digest: ${{ steps.build.outputs.digest }}
    registry: ghcr.io
    repository: ${{ github.repository }}
```

### Sign Container Image by Tags

```yaml
- name: Sign multiple tags
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    image-digest: sha256:abc123...
    image-tags: latest,v1.0.0,stable
    registry: ghcr.io
    repository: myorg/myapp
```

### Sign Artifact File

```yaml
- name: Generate SBOM
  run: cargo cyclonedx --all --format json

- name: Sign SBOM
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    artifact-path: my-app.cdx.json
```

### Sign Both Image and Artifacts

```yaml
- name: Build and push
  id: build
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

- name: Generate SBOM
  uses: your-org/github-actions/rust/generate-sbom@v1

- name: Sign container image
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    image-digest: ${{ steps.build.outputs.digest }}
    registry: ghcr.io
    repository: ${{ github.repository }}

- name: Sign SBOM
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    artifact-path: my-app.cdx.json
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `image-digest` | Docker image digest to sign (e.g., `sha256:abc123...`) | No | N/A |
| `image-tags` | Comma-separated list of image tags to sign (e.g., `latest,v1.0.0`) | No | N/A |
| `artifact-path` | Path to artifact file to sign (e.g., `binary.tar.gz`, `sbom.json`) | No | N/A |
| `registry` | Container registry (e.g., `ghcr.io`, `docker.io`) | No | `ghcr.io` |
| `repository` | Image repository (e.g., `myorg/myapp`) | No | N/A |

## Outputs

| Name | Description |
|------|-------------|
| `signature-bundle` | Path to the signature bundle file (only for artifact signing) |

## How It Works

### Keyless Signing

Cosign uses Sigstore for keyless signing:

1. **OIDC Authentication**: Uses GitHub's OIDC provider to authenticate
2. **Ephemeral Keys**: Generates temporary signing keys
3. **Certificate Issuance**: Sigstore Fulcio issues a certificate
4. **Transparency Log**: Signature recorded in Rekor transparency log
5. **No Key Storage**: No private keys to manage or rotate

### Image Signing Process

1. **Install Cosign**: Installs specified version
2. **Sign by Digest**: Most reliable, uses immutable digest
3. **Sign by Tags**: Optional, signs additional tags
4. **Verify**: Runs smoke test to ensure signature is valid

### Artifact Signing Process

1. **Install Cosign**: Installs specified version
2. **Sign Blob**: Creates signature bundle
3. **Generate Bundle**: Combines signature and certificate
4. **Verify**: Runs smoke test to ensure signature is valid

### Signature Storage

- **Container Images**: Signatures stored in the same registry as the image
- **Artifacts**: Signature bundle saved as `{artifact-path}.bundle`

## Required Permissions

For OIDC-based signing:

```yaml
permissions:
  id-token: write    # Required for OIDC
  packages: write    # Required to push signatures to registry
  contents: read     # Required for checkout
```

## Best Practices

### 1. Always Sign by Digest

Digests are immutable, tags can be overwritten:

```yaml
# ✓ GOOD - Sign by digest
- uses: your-org/github-actions/security/cosign-sign@v1
  with:
    image-digest: ${{ steps.build.outputs.digest }}

# ✗ AVOID - Signing only by tag
- uses: your-org/github-actions/security/cosign-sign@v1
  with:
    image-tags: latest
```

### 2. Sign After Push

Only sign images that are already pushed:

```yaml
- name: Build and push
  uses: docker/build-push-action@v5
  with:
    push: true  # Must be true

- name: Sign (after push)
  uses: your-org/github-actions/security/cosign-sign@v1
```

### 3. Upload Signature Bundles

Preserve signature bundles as artifacts:

```yaml
- name: Sign SBOM
  id: sign
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    artifact-path: sbom.json

- name: Upload signature bundle
  uses: actions/upload-artifact@v4
  with:
    name: signature-bundle
    path: ${{ steps.sign.outputs.signature-bundle }}
```

### 4. Sign All Release Artifacts

For releases, sign everything:

```yaml
- name: Sign container image
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    image-digest: ${{ steps.build.outputs.digest }}

- name: Sign binary
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    artifact-path: my-app

- name: Sign SBOM
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    artifact-path: my-app.cdx.json
```

### 5. Document Verification

Add verification instructions to your README:

```markdown
## Verifying Signatures

Verify container image:
\```bash
cosign verify \
  --certificate-identity-regexp='https://github.com/myorg/myapp' \
  --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
  ghcr.io/myorg/myapp@sha256:abc123...
\```

Verify artifact:
\```bash
cosign verify-blob \
  --bundle my-app.bundle \
  --certificate-identity-regexp='https://github.com/myorg/myapp' \
  --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
  my-app
\```
```

## Troubleshooting

### OIDC Token Error

**Problem**: "failed to get OIDC token"

**Solution**: Add required permissions:
```yaml
permissions:
  id-token: write
  packages: write
```

### Image Not Found

**Problem**: "image not found in registry"

**Solution**: Ensure image is pushed before signing:
```yaml
- uses: docker/build-push-action@v5
  with:
    push: true  # Must be true
```

### Signature Not Stored

**Problem**: Signature created but not found later

**Solution**: Check registry permissions:
```yaml
permissions:
  packages: write  # Required to push signatures
```

### Verification Fails

**Problem**: Smoke test fails with "signature verification failed"

**Solution**: This is expected if:
1. Image was just pushed (wait a few seconds)
2. Registry mirrors are syncing (try again)
3. Network issues (transient, retry)

## Advanced Usage

### Custom Cosign Version

To use a specific Cosign version, modify the action:

```yaml
- name: Install Cosign
  uses: sigstore/cosign-installer@v3.7.0
  with:
    cosign-release: 'v2.3.0'  # Specify version
```

### Sign with Annotations

Add custom annotations to signatures:

```yaml
- name: Sign with metadata
  shell: bash
  env:
    COSIGN_EXPERIMENTAL: "true"
  run: |
    cosign sign --yes \
      -a version=${{ github.ref_name }} \
      -a commit=${{ github.sha }} \
      -a build_time=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
      ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
```

### Verify Signature Manually

```bash
# Verify image signature
cosign verify \
  --certificate-identity-regexp='https://github.com/myorg/myapp' \
  --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
  ghcr.io/myorg/myapp:latest

# Verify artifact signature
cosign verify-blob \
  --bundle artifact.bundle \
  --certificate-identity-regexp='https://github.com/myorg/myapp' \
  --certificate-oidc-issuer='https://token.actions.githubusercontent.com' \
  artifact.tar.gz
```

### Extract Signature Information

```bash
# Get signature details
cosign verify ghcr.io/myorg/myapp:latest | jq .

# Output includes:
# - Certificate details
# - OIDC issuer
# - Repository identity
# - Rekor transparency log entry
```

## Understanding Signature Bundles

### Bundle Structure

Artifact signature bundles contain:

```json
{
  "base64Signature": "...",
  "cert": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
  "rekorBundle": {
    "SignedEntryTimestamp": "...",
    "Payload": {
      "body": "...",
      "integratedTime": 1234567890,
      "logIndex": 12345,
      "logID": "..."
    }
  }
}
```

### Bundle Components

- **base64Signature**: The cryptographic signature
- **cert**: X.509 certificate from Fulcio
- **rekorBundle**: Transparency log entry from Rekor

## Verification Process

### For Container Images

1. Fetch signature from registry
2. Verify certificate chain (Fulcio)
3. Check transparency log (Rekor)
4. Verify signature matches image digest
5. Validate OIDC identity claims

### For Artifacts

1. Read signature bundle
2. Verify certificate chain (Fulcio)
3. Check transparency log (Rekor)
4. Verify signature matches file hash
5. Validate OIDC identity claims

## Security Considerations

### Identity Verification

Cosign verifies:
- **Repository**: Signature came from the correct GitHub repository
- **OIDC Issuer**: GitHub Actions is the issuer
- **Workflow**: (Optional) Specific workflow created the signature

### Transparency

All signatures are logged in Rekor:
- Public transparency log
- Immutable audit trail
- Verifiable by anyone
- No retroactive changes possible

### Certificate Validity

Certificates are:
- Short-lived (10 minutes)
- Bound to OIDC identity
- Verified against Fulcio root
- Timestamped in Rekor

## Performance Tips

### 1. Cache Cosign Binary

The action already caches the Cosign binary via `sigstore/cosign-installer`.

### 2. Sign in Parallel

Sign multiple artifacts in parallel:

```yaml
- name: Sign image (parallel)
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    image-digest: ${{ steps.build.outputs.digest }}

- name: Sign SBOM (parallel)
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    artifact-path: sbom.json
```

### 3. Skip Verification for Speed

To skip the smoke test (not recommended):

Modify the action to remove the verification step.

## Related Actions

- [security/trivy-scan](../trivy-scan/README.md) - Scan before signing
- [rust/generate-sbom](../../rust/generate-sbom/README.md) - Generate SBOMs to sign
- [docker/setup-docker](../../docker/setup-docker/README.md) - Setup Docker environment
- [security/verify-signed-commits](../verify-signed-commits/README.md) - Verify commit signatures

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Requires internet access to Sigstore services
- **Cosign versions**: Tested with v2.4.1
- **Registries**: GHCR, Docker Hub, ACR, GCR, ECR, Quay, Harbor

## Sigstore Resources

- [Cosign Documentation](https://docs.sigstore.dev/cosign/overview/)
- [Sigstore Website](https://www.sigstore.dev/)
- [Rekor Transparency Log](https://rekor.sigstore.dev/)
- [Fulcio Certificate Authority](https://docs.sigstore.dev/fulcio/overview/)

## Examples

### Complete Release Workflow

```yaml
name: Release

on:
  push:
    tags: ['v*']

permissions:
  contents: write
  packages: write
  id-token: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Docker
        uses: your-org/github-actions/docker/setup-docker@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.ref_name }}

      - name: Generate SBOM
        uses: your-org/github-actions/rust/generate-sbom@v1
        with:
          format: both

      - name: Sign container image
        uses: your-org/github-actions/security/cosign-sign@v1
        with:
          image-digest: ${{ steps.build.outputs.digest }}
          image-tags: latest,${{ github.ref_name }}
          registry: ghcr.io
          repository: ${{ github.repository }}

      - name: Sign SBOM (JSON)
        id: sign-sbom-json
        uses: your-org/github-actions/security/cosign-sign@v1
        with:
          artifact-path: my-app.cdx.json

      - name: Sign SBOM (XML)
        id: sign-sbom-xml
        uses: your-org/github-actions/security/cosign-sign@v1
        with:
          artifact-path: my-app.cdx.xml

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            my-app.cdx.json
            my-app.cdx.xml
            ${{ steps.sign-sbom-json.outputs.signature-bundle }}
            ${{ steps.sign-sbom-xml.outputs.signature-bundle }}
```

### Multi-Architecture Release

```yaml
jobs:
  build-and-sign:
    strategy:
      matrix:
        platform:
          - linux/amd64
          - linux/arm64
    steps:
      - name: Build
        id: build
        uses: docker/build-push-action@v5
        with:
          platforms: ${{ matrix.platform }}
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ matrix.platform }}

      - name: Sign
        uses: your-org/github-actions/security/cosign-sign@v1
        with:
          image-digest: ${{ steps.build.outputs.digest }}
          registry: ghcr.io
          repository: ${{ github.repository }}
```
