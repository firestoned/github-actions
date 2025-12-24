# Setup Rust Build Environment

A composite GitHub Action that sets up a complete Rust build environment with intelligent caching for multi-architecture builds, including ARM64 support via cross-compilation.

## Features

- Installs Rust toolchain with specified target support
- Intelligent caching of Rust dependencies using `rust-cache`
- Automatic setup of `cross` for ARM64 Linux and Windows builds
- Binary caching for `cross` to speed up workflow runs
- Docker verification for cross-compilation
- Support for Linux, Windows (MSVC and GNU), and cross-compilation targets
- Configurable cache keys for custom cache isolation

## Usage

### Basic Example (x86_64)

```yaml
- name: Setup Rust build environment
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
```

### ARM64 Linux Cross-Compilation

```yaml
- name: Setup Rust build environment for ARM64 Linux
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-unknown-linux-gnu
    cross-version: v0.2.5
```

### Windows x86_64 Build (MSVC)

```yaml
- name: Setup Rust build environment for Windows
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-pc-windows-msvc
```

### Windows ARM64 Cross-Compilation

```yaml
- name: Setup Rust build environment for Windows ARM64
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-pc-windows-msvc
    cross-version: v0.2.5
```

### Custom Cache Key

```yaml
- name: Setup Rust build environment with custom cache
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
    cache-key: my-feature-branch
```

### Custom Components

```yaml
# Minimal setup without linting tools
- name: Setup Rust (build only)
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
    components: ''

# Add additional components
- name: Setup Rust with extra tools
  uses: firestoned/github-actions/rust/setup-rust-build@v1
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
          - x86_64-pc-windows-msvc
          - x86_64-pc-windows-gnu
          - aarch64-pc-windows-msvc
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust build environment
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: ${{ matrix.target }}

      - name: Build
        run: |
          if [[ "${{ matrix.target }}" == "aarch64-unknown-linux-gnu" || "${{ matrix.target }}" == "aarch64-pc-windows-msvc" ]]; then
            cross build --release --target ${{ matrix.target }}
          else
            cargo build --release --target ${{ matrix.target }}
          fi
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `target` | Rust target triple (e.g., `x86_64-unknown-linux-gnu`, `x86_64-pc-windows-msvc`, `aarch64-unknown-linux-gnu`) | Yes | N/A |
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

### Cross-Compilation Builds (ARM64 and Windows ARM64)

For targets that require cross-compilation (`aarch64-unknown-linux-gnu`, `aarch64-pc-windows-msvc`):

1. Performs all standard build steps
2. Determines if `cross` is needed based on target
3. Checks for cached `cross` binary
4. Installs `cross` if not cached (pinned to specified version)
5. Verifies Docker is available for cross-compilation
6. Caches `cross` binary for future runs

### Cache Strategy

The action uses multiple layers of caching:

- **Rust dependencies**: Cached by `rust-cache` with key: `{target}-{cache-key}`
- **cross binary**: Cached separately with key: `{os}-cross-{version}`

This ensures:
- Fast rebuilds when dependencies haven't changed
- Isolated caches for different targets and branches
- Quick setup for ARM64 builds without reinstalling `cross`

## Supported Targets

### Linux Targets (Native and Cross-Compilation)
- `x86_64-unknown-linux-gnu` - x86_64 Linux with GNU libc (native)
- `x86_64-unknown-linux-musl` - x86_64 Linux with musl libc (native)
- `aarch64-unknown-linux-gnu` - ARM64 Linux with GNU libc (cross-compilation with `cross`)
- `aarch64-unknown-linux-musl` - ARM64 Linux with musl libc (cross-compilation)
- `armv7-unknown-linux-gnueabihf` - 32-bit ARM Linux (cross-compilation)

### Windows Targets
- `x86_64-pc-windows-msvc` - x86_64 Windows with MSVC toolchain (cross-compilation from Linux)
- `x86_64-pc-windows-gnu` - x86_64 Windows with GNU toolchain (cross-compilation from Linux)
- `aarch64-pc-windows-msvc` - ARM64 Windows with MSVC toolchain (cross-compilation with `cross`)

### macOS Targets
- `x86_64-apple-darwin` - x86_64 macOS (native on x86_64 macOS runners)
- `aarch64-apple-darwin` - ARM64 macOS (native on ARM64 macOS runners)

### Automatically Uses Cross

This action automatically installs and configures `cross` for these targets:
- `aarch64-unknown-linux-gnu` - ARM64 Linux
- `aarch64-pc-windows-msvc` - ARM64 Windows

For a complete list of Rust targets, see the [Rust Platform Support](https://doc.rust-lang.org/nightly/rustc/platform-support.html) documentation.

## Best Practices

### 1. Pin cross Version

Always specify a `cross-version` for reproducible builds:

```yaml
- uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-unknown-linux-gnu
    cross-version: v0.2.5
```

### 2. Use Cache Keys for Branch Isolation

For long-lived feature branches with different dependencies:

```yaml
- uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
    cache-key: ${{ github.ref_name }}
```

### 3. Combine with Build Actions

This action is designed to work seamlessly with the `rust/build-binary` action:

```yaml
- name: Setup Rust environment
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: ${{ matrix.target }}

- name: Build binary
  uses: firestoned/github-actions/rust/build-binary@v1
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
        uses: firestoned/github-actions/rust/setup-rust-build@v1
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

- uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu
```
