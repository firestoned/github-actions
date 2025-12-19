# Setup Docker Build Environment

A composite GitHub Action that sets up Docker Buildx for multi-architecture builds and authenticates with GitHub Container Registry (GHCR) in a single step.

## Features

- Sets up Docker Buildx for advanced build features
- Multi-architecture build support (amd64, arm64, etc.)
- Automatic authentication to GitHub Container Registry
- Build caching support
- Docker layer caching
- Cross-platform image builds
- Single-step setup for Docker workflows

## Usage

### Basic Example

```yaml
- name: Setup Docker
  uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Build and Push Image

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Setup Docker
        uses: your-org/github-actions/docker/setup-docker@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

### Multi-Architecture Build

```yaml
- name: Setup Docker
  uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}

- name: Build multi-arch image
  uses: docker/build-push-action@v5
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ghcr.io/${{ github.repository }}:latest
```

### With Build Cache

```yaml
- name: Setup Docker
  uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}

- name: Build with cache
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
    cache-from: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache
    cache-to: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache,mode=max
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `github_token` | GitHub token for GHCR authentication (use `${{ secrets.GITHUB_TOKEN }}`) | Yes | N/A |

## How It Works

### Setup Process

1. **Set up Docker Buildx**: Installs and configures Buildx
   - Enables advanced Docker features
   - Supports multi-architecture builds
   - Provides build caching

2. **Authenticate to GHCR**: Logs into GitHub Container Registry
   - Uses `github.actor` as username
   - Uses provided token for authentication
   - Credentials stored for subsequent Docker commands

### What is Docker Buildx?

Docker Buildx is a CLI plugin that extends Docker with:
- Multi-platform image builds
- Advanced caching strategies
- Parallel build execution
- Build secrets and SSH forwarding
- Exporters for different output formats

## Required Permissions

For pushing to GHCR:

```yaml
permissions:
  contents: read   # Read repository contents
  packages: write  # Push to GHCR
```

## Best Practices

### 1. Use Secrets Token

Always use `secrets.GITHUB_TOKEN`:

```yaml
with:
  github_token: ${{ secrets.GITHUB_TOKEN }}
```

**Never** use personal access tokens unless needed for private base images.

### 2. Tag Images Properly

Use meaningful tags:

```yaml
tags: |
  ghcr.io/${{ github.repository }}:latest
  ghcr.io/${{ github.repository }}:${{ github.sha }}
  ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

### 3. Enable Layer Caching

For faster builds:

```yaml
cache-from: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache
cache-to: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache,mode=max
```

### 4. Use Multi-Stage Builds

In your Dockerfile:

```dockerfile
# Build stage
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

# Runtime stage
FROM debian:bookworm-slim
COPY --from=builder /app/target/release/app /usr/local/bin/
CMD ["app"]
```

### 5. Build Multi-Arch for Production

```yaml
platforms: linux/amd64,linux/arm64
```

Supports:
- AWS Graviton (ARM64)
- Apple Silicon (ARM64)
- Traditional servers (AMD64)

## Troubleshooting

### Authentication Failed

**Problem**: "authentication required" or "denied: permission_denied"

**Solution**: Check permissions:
```yaml
permissions:
  packages: write  # Required!
```

### Buildx Not Available

**Problem**: "buildx: command not found"

**Solution**: This action installs Buildx automatically. Ensure you're using this action before running Docker commands.

### Multi-Arch Build Fails

**Problem**: "no match for platform in manifest"

**Solution**: Install QEMU for emulation:
```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3

- name: Setup Docker
  uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Push Permission Denied

**Problem**: "denied: permission_denied" when pushing

**Solution**: Verify:
1. `packages: write` permission is set
2. Repository allows package creation
3. Token has not expired
4. Image name matches repository format: `ghcr.io/OWNER/REPO`

### Build Cache Not Working

**Problem**: Builds always run from scratch

**Solution**: Ensure cache tags are consistent:
```yaml
cache-from: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache
cache-to: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache,mode=max
```

## Advanced Usage

### Custom Registry

To use a different registry:

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Login to custom registry
  uses: docker/login-action@v3
  with:
    registry: myregistry.example.com
    username: ${{ secrets.REGISTRY_USERNAME }}
    password: ${{ secrets.REGISTRY_PASSWORD }}
```

### Multiple Registries

Login to multiple registries:

```yaml
- uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}

- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Build Metadata

Add metadata to images:

```yaml
- name: Setup Docker
  uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}

- name: Extract metadata
  id: meta
  uses: docker/metadata-action@v5
  with:
    images: ghcr.io/${{ github.repository }}
    tags: |
      type=ref,event=branch
      type=ref,event=pr
      type=semver,pattern={{version}}
      type=sha

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

### Build Secrets

Pass secrets to builds:

```yaml
- uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}

- name: Build with secrets
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    secrets: |
      GIT_AUTH_TOKEN=${{ secrets.GIT_TOKEN }}
    tags: ghcr.io/${{ github.repository }}:latest
```

In Dockerfile:
```dockerfile
RUN --mount=type=secret,id=GIT_AUTH_TOKEN \
    GIT_TOKEN=$(cat /run/secrets/GIT_AUTH_TOKEN) && \
    git clone https://${GIT_TOKEN}@github.com/private/repo.git
```

### Local Testing

Test builds locally without pushing:

```yaml
- uses: your-org/github-actions/docker/setup-docker@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}

- name: Build only (no push)
  uses: docker/build-push-action@v5
  with:
    context: .
    load: true
    tags: myapp:test

- name: Test image
  run: |
    docker run --rm myapp:test --version
```

## Docker Buildx Features

### Multi-Platform Builds

Build for multiple architectures in one command:

```yaml
platforms: linux/amd64,linux/arm64,linux/arm/v7
```

### Build Drivers

Buildx supports multiple drivers:

| Driver | Use Case | Multi-Arch | Remote |
|--------|----------|------------|--------|
| docker | Default, single-arch | No | No |
| docker-container | Isolated, multi-arch | Yes | No |
| kubernetes | K8s deployments | Yes | Yes |
| remote | Remote buildkit | Yes | Yes |

Default driver is `docker-container` (best for CI/CD).

### Cache Backends

Multiple caching strategies:

```yaml
# Registry cache (recommended for CI)
cache-from: type=registry,ref=ghcr.io/myapp:buildcache
cache-to: type=registry,ref=ghcr.io/myapp:buildcache,mode=max

# GitHub Actions cache
cache-from: type=gha
cache-to: type=gha,mode=max

# Local cache
cache-from: type=local,src=/tmp/.buildx-cache
cache-to: type=local,dest=/tmp/.buildx-cache,mode=max
```

### Output Formats

Export builds in different formats:

```yaml
# Docker image (default)
outputs: type=docker

# OCI tarball
outputs: type=oci,dest=image.tar

# Push to registry
outputs: type=registry

# Local filesystem
outputs: type=local,dest=./output
```

## Performance Tips

### 1. Use Layer Caching

```yaml
cache-from: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache
cache-to: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache,mode=max
```

Reduces build times by 50-90% for incremental changes.

### 2. Optimize Dockerfile

```dockerfile
# Cache dependencies separately from code
COPY Cargo.toml Cargo.lock ./
RUN cargo fetch

# Then copy and build code
COPY . .
RUN cargo build --release
```

### 3. Use GitHub Actions Cache

```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

Free caching up to 10GB per repository.

### 4. Parallel Builds

Build multiple images in parallel:

```yaml
strategy:
  matrix:
    image: [app, worker, cron]
steps:
  - uses: your-org/github-actions/docker/setup-docker@v1
    with:
      github_token: ${{ secrets.GITHUB_TOKEN }}

  - uses: docker/build-push-action@v5
    with:
      context: .
      file: Dockerfile.${{ matrix.image }}
      push: true
      tags: ghcr.io/${{ github.repository }}/${{ matrix.image }}:latest
```

## Security Considerations

### Image Scanning

Always scan images before pushing:

```yaml
- name: Build image
  uses: docker/build-push-action@v5
  with:
    context: .
    load: true
    tags: myapp:test

- name: Scan with Trivy
  uses: your-org/github-actions/security/trivy-scan@v1
  with:
    image-ref: myapp:test

- name: Push if scan passes
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ghcr.io/${{ github.repository }}:latest
```

### Image Signing

Sign images with Cosign:

```yaml
- name: Build and push
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

### Minimal Base Images

Use minimal base images:

```dockerfile
# ✓ GOOD - Minimal attack surface
FROM gcr.io/distroless/static-debian12

# ✓ GOOD - Alpine is small
FROM alpine:3.19

# ✗ AVOID - Large attack surface
FROM ubuntu:latest
```

## Related Actions

- [security/trivy-scan](../../security/trivy-scan/README.md) - Scan Docker images
- [security/cosign-sign](../../security/cosign-sign/README.md) - Sign Docker images
- [rust/setup-rust-build](../../rust/setup-rust-build/README.md) - Setup Rust builds

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Requires Docker 20.10+
- **Buildx versions**: Tested with latest from docker/setup-buildx-action@v3
- **Registries**: GHCR, Docker Hub, ACR, GCR, ECR, Quay, Harbor

## Resources

- [Docker Buildx Documentation](https://docs.docker.com/buildx/working-with-buildx/)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Multi-Platform Images](https://docs.docker.com/build/building/multi-platform/)
- [Build Caching](https://docs.docker.com/build/cache/)

## Examples

### Complete CI/CD Pipeline

```yaml
name: Docker Build

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

permissions:
  contents: read
  packages: write
  id-token: write
  security-events: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Docker
        uses: your-org/github-actions/docker/setup-docker@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha

      - name: Build and push
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Scan image
        if: github.event_name != 'pull_request'
        uses: your-org/github-actions/security/trivy-scan@v1
        with:
          image-ref: ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}

      - name: Sign image
        if: github.event_name != 'pull_request'
        uses: your-org/github-actions/security/cosign-sign@v1
        with:
          image-digest: ${{ steps.build.outputs.digest }}
          registry: ghcr.io
          repository: ${{ github.repository }}
```

### Development Workflow

```yaml
name: Dev Build

on:
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Docker
        uses: your-org/github-actions/docker/setup-docker@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

      - name: Build (no push)
        uses: docker/build-push-action@v5
        with:
          context: .
          load: true
          tags: myapp:pr-${{ github.event.pull_request.number }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Test image
        run: |
          docker run --rm myapp:pr-${{ github.event.pull_request.number }} --version
          docker run --rm myapp:pr-${{ github.event.pull_request.number }} --help
```
