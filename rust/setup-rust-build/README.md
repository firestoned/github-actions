# Setup Rust Build Environment

A composite GitHub Action that sets up a complete Rust build environment with intelligent caching for multi-architecture builds, including ARM64 support via cross-compilation.

## Features

- Installs Rust toolchain with specified target support
- Intelligent caching of Rust dependencies using `rust-cache`
- Automatic setup of `cross` for ARM64 builds
- Binary caching for `cross` to speed up workflow runs
- Docker verification for cross-compilation
- Configurable cache keys for custom cache isolation

## Usage

### Basic Example (x86_64)

```yaml
- name: Setup Rust build environment
  uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
```

### ARM64 Cross-Compilation

```yaml
- name: Setup Rust build environment for ARM64
  uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-unknown-linux-gnu
    cross-version: v0.2.5
```

### Custom Cache Key

```yaml
- name: Setup Rust build environment with custom cache
  uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
    cache-key: my-feature-branch
```

### Custom Components

```yaml
# Minimal setup without linting tools
- name: Setup Rust (build only)
  uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
    components: ''

# Add additional components
- name: Setup Rust with extra tools
  uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
    components: 'rustfmt, clippy, llvm-tools-preview'
```

### Multi-Target Matrix Build

```yaml
jobs:
  build:
    strategy:
      matrix:
        target:
          - x86_64-unknown-linux-gnu
          - aarch64-unknown-linux-gnu
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust build environment
        uses: your-org/github-actions/rust/setup-rust-build@v1
        with:
          target: ${{ matrix.target }}

      - name: Build
        run: cargo build --release --target ${{ matrix.target }}
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `target` | Rust target triple (e.g., `x86_64-unknown-linux-gnu`, `aarch64-unknown-linux-gnu`) | Yes | N/A |
| `cache-key` | Additional cache key component for isolating caches by branch or feature | No | `''` |
| `cross-version` | Version of `cross` to install for ARM64 builds (e.g., `v0.2.5`) | No | `v0.2.5` |
| `components` | Comma-separated list of Rust components to install (e.g., `rustfmt, clippy`) | No | `rustfmt, clippy` |

## How It Works

### Standard Builds (x86_64)

1. Installs Rust toolchain using `dtolnay/rust-toolchain@stable`
2. Adds the specified target to the toolchain
3. Installs components (default: `rustfmt, clippy`)
4. Caches Rust dependencies using `Swatinem/rust-cache@v2`
5. Uses target and cache-key for cache isolation

### ARM64 Builds (aarch64)

1. Performs all standard build steps
2. Checks for cached `cross` binary
3. Installs `cross` if not cached (pinned to specified version)
4. Verifies Docker is available for cross-compilation
5. Caches `cross` binary for future runs

### Cache Strategy

The action uses multiple layers of caching:

- **Rust dependencies**: Cached by `rust-cache` with key: `{target}-{cache-key}`
- **cross binary**: Cached separately with key: `{os}-cross-{version}`

This ensures:
- Fast rebuilds when dependencies haven't changed
- Isolated caches for different targets and branches
- Quick setup for ARM64 builds without reinstalling `cross`

## Supported Targets

### Tier 1 Targets (Native Builds)
- `x86_64-unknown-linux-gnu` - 64-bit Linux (GNU)
- `x86_64-unknown-linux-musl` - 64-bit Linux (musl)
- `x86_64-apple-darwin` - 64-bit macOS
- `x86_64-pc-windows-msvc` - 64-bit Windows (MSVC)

### Tier 2 Targets (Cross-Compilation)
- `aarch64-unknown-linux-gnu` - 64-bit ARM Linux (GNU)
- `aarch64-unknown-linux-musl` - 64-bit ARM Linux (musl)
- `armv7-unknown-linux-gnueabihf` - 32-bit ARM Linux

For a complete list, see the [Rust Platform Support](https://doc.rust-lang.org/nightly/rustc/platform-support.html) documentation.

## Best Practices

### 1. Pin cross Version

Always specify a `cross-version` for reproducible builds:

```yaml
- uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-unknown-linux-gnu
    cross-version: v0.2.5
```

### 2. Use Cache Keys for Branch Isolation

For long-lived feature branches with different dependencies:

```yaml
- uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
    cache-key: ${{ github.ref_name }}
```

### 3. Combine with Build Actions

This action is designed to work seamlessly with the `rust/build-binary` action:

```yaml
- name: Setup Rust environment
  uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: ${{ matrix.target }}

- name: Build binary
  uses: your-org/github-actions/rust/build-binary@v1
  with:
    target: ${{ matrix.target }}
```

## Troubleshooting

### cross Installation Fails

**Problem**: `cross` installation times out or fails

**Solution**: The action caches `cross` binaries. If installation fails:
1. Check the `cross-version` is valid
2. Verify the cross repository is accessible
3. Try clearing the cache and re-running

### Docker Not Available for ARM64

**Problem**: ARM64 build fails with "Docker not found"

**Solution**: Ensure your runner has Docker installed:
```yaml
- name: Set up Docker
  uses: docker/setup-buildx-action@v3
```

### Cache Not Restoring

**Problem**: Dependencies rebuild on every run

**Solution**:
1. Check that `Cargo.lock` is committed to your repository
2. Verify the `cache-key` is consistent across runs
3. Ensure `Cargo.toml` hasn't changed (triggers cache invalidation)

### Wrong Toolchain Version

**Problem**: Build uses incorrect Rust version

**Solution**: This action uses `stable` by default. For specific versions, use:
```yaml
- uses: dtolnay/rust-toolchain@stable
  with:
    toolchain: 1.75.0
    targets: ${{ matrix.target }}
```

## Performance Tips

### Optimize Cache Hit Rate

1. **Commit Cargo.lock**: Ensures consistent dependency versions
2. **Stable dependencies**: Minimize changes to `Cargo.toml`
3. **Branch caching**: Use `cache-key` to isolate development branches

### Reduce Build Times

1. **Use rust-cache**: Already included, handles incremental compilation
2. **Parallel builds**: Use matrix strategy for multiple targets
3. **Cache cross**: Already included, saves ~2-3 minutes per ARM64 build

## Related Actions

- [rust/build-binary](../build-binary/README.md) - Build Rust binaries (uses this action)
- [rust/cache-cargo](../../cache/cache-cargo/README.md) - Alternative manual caching
- [rust/security-scan](../security-scan/README.md) - Security vulnerability scanning
- [rust/generate-sbom](../generate-sbom/README.md) - Generate Software Bill of Materials

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: Compatible with all GitHub-hosted runners
- **Self-hosted runners**: Requires Docker for ARM64 cross-compilation
- **Rust versions**: Works with stable, beta, and nightly toolchains
- **cross versions**: Tested with v0.2.5 and later

## Examples

### Complete CI Workflow

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    strategy:
      matrix:
        target:
          - x86_64-unknown-linux-gnu
          - aarch64-unknown-linux-gnu
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: your-org/github-actions/rust/setup-rust-build@v1
        with:
          target: ${{ matrix.target }}
          cache-key: ${{ github.ref_name }}

      - name: Run tests
        run: cargo test --target ${{ matrix.target }}

      - name: Build release
        run: cargo build --release --target ${{ matrix.target }}
```

### With Custom Toolchain

```yaml
- uses: dtolnay/rust-toolchain@stable
  with:
    toolchain: nightly
    targets: x86_64-unknown-linux-gnu

- uses: your-org/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
```
