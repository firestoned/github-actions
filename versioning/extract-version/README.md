# Extract Version Information

**Extracts version, tag name, and image tags for consistent use across workflows**

A composite GitHub Action that provides standardized version extraction for different workflow contexts (main branch, pull requests, and releases). Ensures consistent versioning and tagging across your entire CI/CD pipeline, with support for image variants like distroless builds.

## Features

- **Three workflow types** - Specialized handling for main, PR, and release workflows
- **Consistent versioning** - Single source of truth for version information across all jobs
- **Image variant support** - Built-in support for distroless and other image variants
- **Semantic versioning** - Proper semver for releases, dated versions for main branch
- **PR-specific tags** - Clean `pr-NUMBER` format for pull request builds
- **Short SHA extraction** - 7-character commit SHA for tracking
- **Repository-agnostic** - Works with any repository, not hardcoded

## Usage

### Basic Usage - Main Branch

```yaml
- name: Extract version
  id: version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: main

- name: Use version
  run: |
    echo "Version: ${{ steps.version.outputs.version }}"
    echo "Tag: ${{ steps.version.outputs.tag-name }}"
    echo "Image: ${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}"
```

**Output:**
```
Version: 0.0.0-main.2025.12.17.42
Tag: main-2025.12.17
Image: owner/myapp:main-2025.12.17
```

### Pull Request Workflow

```yaml
- name: Extract version
  id: version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: pr
    pr-number: ${{ github.event.pull_request.number }}

- name: Build PR image
  run: docker build -t ${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }} .
```

**Output:**
```
Version: pr-42
Tag: pr-42
Image: owner/myapp:pr-42
```

### Release Workflow

```yaml
- name: Extract version
  id: version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: release
    release-tag: ${{ github.ref_name }}

- name: Build release image
  run: docker build -t ${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }} .
```

**Output:**
```
Version: 0.2.0
Tag: v0.2.0
Image: owner/myapp:v0.2.0
```

### Distroless Image Variant

```yaml
- name: Extract version for distroless
  id: version-distroless
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: main
    image-suffix: -distroless

- name: Build distroless image
  run: |
    docker build -f Dockerfile.distroless \
      -t ${{ steps.version-distroless.outputs.image-repository }}:${{ steps.version-distroless.outputs.image-tag }} .
```

**Output:**
```
Image Repository: owner/myapp-distroless
Image Tag: main-2025.12.17
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `repository` | Repository name (e.g., `owner/myapp`) - used for image repository naming | Yes | N/A |
| `workflow-type` | Workflow type: `main`, `pr`, or `release` | Yes | N/A |
| `pr-number` | PR number (only for `pr` workflow) | No | N/A |
| `release-tag` | Release tag name (only for `release` workflow) | No | N/A |
| `image-suffix` | Image suffix for variant (e.g., `""` or `"-distroless"`) | No | `""` |

## Outputs

| Name | Description | Example |
|------|-------------|---------|
| `version` | Semantic version | `1.2.3` or `0.0.0-main.2025.12.17.42` or `pr-42` |
| `tag-name` | Git tag name | `v1.2.3` or `main-2025.12.17` or `pr-42` |
| `image-tag` | Docker image tag | `v1.2.3` or `main-2025.12.17` or `pr-42` |
| `image-repository` | Docker repository name | `owner/myapp` or `owner/myapp-distroless` |
| `short-sha` | Short 7-character SHA | `a1b2c3d` |

## How It Works

### Workflow Type: `main`

For commits to the main branch:

1. **Version format**: `0.0.0-main.YYYY.MM.DD.RUN_NUMBER`
   - Example: `0.0.0-main.2025.12.17.42`
2. **Tag format**: `main-YYYY.MM.DD`
   - Example: `main-2025.12.17`
3. **Image tag**: Same as tag name
   - Example: `main-2025.12.17`

This ensures:
- Daily builds are versioned consistently
- Same date = same tag (builds on same day overwrite)
- Run number included in version for uniqueness
- Clear indication this is a development build

### Workflow Type: `pr`

For pull request builds:

1. **Version format**: `pr-NUMBER`
   - Example: `pr-42`
2. **Tag format**: Same as version
   - Example: `pr-42`
3. **Image tag**: Same as version
   - Example: `pr-42`

This ensures:
- Clear indication this is a PR build
- Easy to identify which PR an artifact came from
- No date complexity needed
- Simple cleanup of PR artifacts

### Workflow Type: `release`

For release tags:

1. **Version format**: Stripped `v` prefix from tag
   - Input: `v0.2.0` → Output: `0.2.0`
2. **Tag format**: Original tag with `v` prefix
   - Example: `v0.2.0`
3. **Image tag**: Original tag with `v` prefix
   - Example: `v0.2.0`

This ensures:
- Proper semantic versioning
- Clear release identification
- Compatible with standard Git tag conventions

### Image Suffix Feature

When `image-suffix` is provided:

- **Base repository**: `owner/myapp`
- **With suffix `-distroless`**: `owner/myapp-distroless`

This allows building multiple image variants with different repository names:
```yaml
# Standard image
owner/myapp:v0.2.0

# Distroless variant
owner/myapp-distroless:v0.2.0
```

## Complete Workflow Examples

### Main Branch Build

```yaml
name: Main Branch Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extract version
        id: version
        uses: firestoned/github-actions/versioning/extract-version@v1
        with:
          repository: ${{ github.repository }}
          workflow-type: main

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}
            ghcr.io/${{ steps.version.outputs.image-repository }}:latest

      - name: Tag Git commit
        run: |
          git tag ${{ steps.version.outputs.tag-name }}
          git push origin ${{ steps.version.outputs.tag-name }}
```

### Pull Request Build

```yaml
name: PR Build

on:
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extract version
        id: version
        uses: firestoned/github-actions/versioning/extract-version@v1
        with:
          repository: ${{ github.repository }}
          workflow-type: pr
          pr-number: ${{ github.event.pull_request.number }}

      - name: Build and push PR image
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}

      - name: Comment PR with image tag
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🐳 PR image built: \`ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}\``
            })
```

### Release Build

```yaml
name: Release

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extract version
        id: version
        uses: firestoned/github-actions/versioning/extract-version@v1
        with:
          repository: ${{ github.repository }}
          workflow-type: release
          release-tag: ${{ github.ref_name }}

      - name: Build and push release
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}
            ghcr.io/${{ steps.version.outputs.image-repository }}:latest

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          name: Release ${{ steps.version.outputs.version }}
          tag_name: ${{ steps.version.outputs.tag-name }}
          body: |
            ## Release ${{ steps.version.outputs.version }}

            Docker Image: `ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}`
            SHA: `${{ steps.version.outputs.short-sha }}`
```

### Multi-Variant Build

```yaml
name: Multi-Variant Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        variant:
          - { suffix: '', dockerfile: 'Dockerfile' }
          - { suffix: '-distroless', dockerfile: 'Dockerfile.distroless' }
    steps:
      - uses: actions/checkout@v4

      - name: Extract version
        id: version
        uses: firestoned/github-actions/versioning/extract-version@v1
        with:
          repository: ${{ github.repository }}
          workflow-type: main
          image-suffix: ${{ matrix.variant.suffix }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          file: ${{ matrix.variant.dockerfile }}
          push: true
          tags: ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}
```

## Migration from Hardcoded Version

### Before (Hardcoded)

```yaml
# Old hardcoded approach - NOT recommended
- name: Set version
  id: version
  run: |
    echo "image=ghcr.io/firestoned/bindy:main-$(date +%Y.%m.%d)" >> $GITHUB_OUTPUT
```

**Problems:**
- Hardcoded repository name
- Inconsistent across workflows
- Manual date formatting
- No variant support
- No proper versioning for releases

### After (Parameterized)

```yaml
# New parameterized approach - RECOMMENDED
- name: Extract version
  id: version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: main

- name: Build
  run: docker build -t ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }} .
```

**Benefits:**
- Repository name from context
- Consistent across all workflows
- Standardized date format
- Support for variants
- Proper handling of releases and PRs

## Best Practices

### 1. Use Repository from Context

Always use `${{ github.repository }}` instead of hardcoding:

```yaml
# ✅ GOOD - Dynamic repository
with:
  repository: ${{ github.repository }}

# ❌ BAD - Hardcoded repository
with:
  repository: firestoned/myapp
```

### 2. Store Version in Job Output

Share version across jobs:

```yaml
jobs:
  version:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.extract.outputs.version }}
      image-tag: ${{ steps.extract.outputs.image-tag }}
    steps:
      - name: Extract version
        id: extract
        uses: firestoned/github-actions/versioning/extract-version@v1
        with:
          repository: ${{ github.repository }}
          workflow-type: ${{ github.event_name == 'pull_request' && 'pr' || 'main' }}
          pr-number: ${{ github.event.pull_request.number }}

  build:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building version ${{ needs.version.outputs.version }}"
```

### 3. Conditional Workflow Type

Determine workflow type dynamically:

```yaml
- name: Determine workflow type
  id: workflow
  run: |
    if [ "${{ github.event_name }}" = "pull_request" ]; then
      echo "type=pr" >> $GITHUB_OUTPUT
    elif [ "${{ github.ref }}" = "refs/heads/main" ]; then
      echo "type=main" >> $GITHUB_OUTPUT
    else
      echo "type=release" >> $GITHUB_OUTPUT
    fi

- name: Extract version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: ${{ steps.workflow.outputs.type }}
    pr-number: ${{ github.event.pull_request.number }}
    release-tag: ${{ github.ref_name }}
```

### 4. Use All Outputs

Take advantage of all available outputs:

```yaml
- name: Extract version
  id: version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: main

- name: Build metadata
  run: |
    echo "Version: ${{ steps.version.outputs.version }}"
    echo "Git Tag: ${{ steps.version.outputs.tag-name }}"
    echo "Image: ${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}"
    echo "Commit: ${{ steps.version.outputs.short-sha }}"

- name: Create labels
  run: |
    docker build \
      --label "version=${{ steps.version.outputs.version }}" \
      --label "git.tag=${{ steps.version.outputs.tag-name }}" \
      --label "git.sha=${{ steps.version.outputs.short-sha }}" \
      -t ${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }} .
```

## Troubleshooting

### Invalid Workflow Type Error

**Problem**: `Error: Invalid workflow-type 'xyz'. Must be: main, pr, or release`

**Solution**: Ensure `workflow-type` is exactly one of: `main`, `pr`, or `release`

```yaml
# ✅ CORRECT
workflow-type: main

# ❌ WRONG
workflow-type: master  # Use 'main' instead
workflow-type: pull_request  # Use 'pr' instead
```

### Missing PR Number

**Problem**: PR workflow but no PR number provided

**Solution**: Always provide `pr-number` for PR workflows:

```yaml
- uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: pr
    pr-number: ${{ github.event.pull_request.number }}  # Required!
```

### Missing Release Tag

**Problem**: Release workflow but no release tag provided

**Solution**: Provide `release-tag` for release workflows:

```yaml
- uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: release
    release-tag: ${{ github.ref_name }}  # Required!
```

### Image Suffix Not Applied

**Problem**: Distroless variant not getting suffix

**Solution**: Verify `image-suffix` includes the hyphen:

```yaml
# ✅ CORRECT
image-suffix: -distroless

# ❌ WRONG
image-suffix: distroless  # Missing hyphen
```

## Advanced Usage

### Dynamic Variant Suffix

```yaml
- name: Determine variant
  id: variant
  run: |
    if [ -f "Dockerfile.distroless" ]; then
      echo "suffix=-distroless" >> $GITHUB_OUTPUT
    else
      echo "suffix=" >> $GITHUB_OUTPUT
    fi

- name: Extract version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: main
    image-suffix: ${{ steps.variant.outputs.suffix }}
```

### Multi-Architecture Builds

```yaml
- name: Extract version
  id: version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: main

- name: Build multi-arch
  uses: docker/build-push-action@v5
  with:
    platforms: linux/amd64,linux/arm64
    push: true
    tags: |
      ghcr.io/${{ steps.version.outputs.image-repository }}:${{ steps.version.outputs.image-tag }}
      ghcr.io/${{ steps.version.outputs.image-repository }}:latest
```

### Reusable Workflow

```yaml
# .github/workflows/version-extract.yml
name: Extract Version

on:
  workflow_call:
    inputs:
      workflow-type:
        required: true
        type: string
    outputs:
      version:
        value: ${{ jobs.extract.outputs.version }}
      image-tag:
        value: ${{ jobs.extract.outputs.image-tag }}

jobs:
  extract:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
      image-tag: ${{ steps.version.outputs.image-tag }}
    steps:
      - name: Extract version
        id: version
        uses: firestoned/github-actions/versioning/extract-version@v1
        with:
          repository: ${{ github.repository }}
          workflow-type: ${{ inputs.workflow-type }}
          pr-number: ${{ github.event.pull_request.number }}
          release-tag: ${{ github.ref_name }}
```

## Related Actions

- [docker/setup-docker](../../docker/setup-docker/README.md) - Setup Docker build environment
- [security/cosign-sign](../../security/cosign-sign/README.md) - Sign images with version tags
- [security/trivy-scan](../../security/trivy-scan/README.md) - Scan versioned images

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Yes (no special requirements)
- **Operating Systems**: Linux, macOS, Windows
- **Git**: Any version
- **Date command**: Standard Unix `date` command

## Version Format Reference

| Workflow Type | Version | Tag Name | Image Tag |
|---------------|---------|----------|-----------|
| `main` | `0.0.0-main.2025.12.17.42` | `main-2025.12.17` | `main-2025.12.17` |
| `pr` | `pr-42` | `pr-42` | `pr-42` |
| `release` | `0.2.0` | `v0.2.0` | `v0.2.0` |

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Contributing

Contributions welcome! This action is designed to be:
- Repository-agnostic (works with any repo)
- Flexible (supports multiple workflow types)
- Extensible (easy to add new variant suffixes)

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.
