# Build Rust Binary

A composite GitHub Action that intelligently builds Rust binaries for specified target architectures, automatically using `cargo` for native builds and `cross` for cross-compilation.

## Features

- Automatic selection of build tool (`cargo` vs `cross`)
- Native builds for x86_64 using standard `cargo`
- Cross-compilation for ARM64 using `cross`
- Verbose output for debugging build issues
- Release profile optimization
- Seamless integration with `rust/setup-rust-build`

## Usage

### Basic Example (x86_64)

```yaml
- name: Setup Rust environment
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: x86_64-unknown-linux-gnu

- name: Build binary
  uses: firestoned/github-actions/rust/build-binary@v1
  with:
    target: x86_64-unknown-linux-gnu
```

### ARM64 Cross-Compilation

```yaml
- name: Setup Rust environment
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-unknown-linux-gnu

- name: Build binary
  uses: firestoned/github-actions/rust/build-binary@v1
  with:
    target: aarch64-unknown-linux-gnu
```

### Multi-Architecture Matrix Build

```yaml
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

      - name: Build
        uses: firestoned/github-actions/rust/build-binary@v1
        with:
          target: ${{ matrix.target }}

      - name: Upload binary
        uses: actions/upload-artifact@v4
        with:
          name: binary-${{ matrix.target }}
          path: target/${{ matrix.target }}/release/my-app
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `target` | Rust target triple (e.g., `x86_64-unknown-linux-gnu`, `aarch64-unknown-linux-gnu`) | Yes | N/A |

## How It Works

### Decision Logic

The action automatically chooses the appropriate build tool based on the target:

| Target | Build Tool | Reason |
|--------|------------|--------|
| `x86_64-unknown-linux-gnu` | `cargo` | Native compilation on x86_64 runners |
| `aarch64-unknown-linux-gnu` | `cross` | Cross-compilation from x86_64 to ARM64 |

### Build Process

#### For x86_64 (Native Build)
```bash
cargo build --release --target x86_64-unknown-linux-gnu --verbose
```

#### For ARM64 (Cross-Compilation)
```bash
cross build --release --target aarch64-unknown-linux-gnu --verbose
```

Both commands:
- Use `--release` for optimized production builds
- Use `--target` for explicit target specification
- Use `--verbose` for detailed build logs

## Output Location

Built binaries are located at:
```
target/{target}/release/{binary-name}
```

For example:
- `target/x86_64-unknown-linux-gnu/release/my-app`
- `target/aarch64-unknown-linux-gnu/release/my-app`

## Best Practices

### 1. Always Use with setup-rust-build

This action requires the build environment to be set up first:

```yaml
- name: Setup Rust
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: ${{ matrix.target }}

- name: Build
  uses: firestoned/github-actions/rust/build-binary@v1
  with:
    target: ${{ matrix.target }}
```

### 2. Match Targets Between Setup and Build

Ensure the same target is used in both actions:

```yaml
env:
  BUILD_TARGET: x86_64-unknown-linux-gnu

steps:
  - uses: firestoned/github-actions/rust/setup-rust-build@v1
    with:
      target: ${{ env.BUILD_TARGET }}

  - uses: firestoned/github-actions/rust/build-binary@v1
    with:
      target: ${{ env.BUILD_TARGET }}
```

### 3. Upload Artifacts with Correct Paths

Use the standard Rust target directory structure:

```yaml
- name: Upload binary
  uses: actions/upload-artifact@v4
  with:
    name: binary-${{ matrix.target }}
    path: target/${{ matrix.target }}/release/my-app
```

## Troubleshooting

### Build Fails with "cargo: not found"

**Problem**: Rust toolchain not installed

**Solution**: Ensure `rust/setup-rust-build` runs first:
```yaml
- uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: ${{ matrix.target }}
```

### Build Fails with "cross: not found"

**Problem**: `cross` not installed for ARM64 build

**Solution**: The `rust/setup-rust-build` action installs `cross` for ARM64 targets. Ensure it runs before this action.

### Binary Not Found After Build

**Problem**: Artifact upload fails with "path not found"

**Solution**: Check the binary name in `Cargo.toml`:
```toml
[package]
name = "my-app"  # Binary will be named "my-app"
```

Then use the correct path:
```yaml
path: target/${{ matrix.target }}/release/my-app
```

### Build Succeeds but Binary Doesn't Run

**Problem**: Binary built for wrong architecture

**Solution**: Verify the target matches your deployment environment:
- Linux servers: `x86_64-unknown-linux-gnu` or `x86_64-unknown-linux-musl`
- ARM servers: `aarch64-unknown-linux-gnu`
- macOS: `x86_64-apple-darwin` or `aarch64-apple-darwin`

### Cross-Compilation Fails for ARM64

**Problem**: `cross` build fails with Docker errors

**Solution**: Ensure Docker is available on the runner:
```yaml
- name: Setup Docker
  uses: docker/setup-buildx-action@v3
```

## Performance Tips

### 1. Use Caching

The `rust/setup-rust-build` action handles caching automatically. Ensure you use it:

```yaml
- uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: ${{ matrix.target }}
    cache-key: ${{ github.ref_name }}
```

### 2. Incremental Compilation

For local development, consider enabling incremental compilation (not recommended for CI):

```yaml
env:
  CARGO_INCREMENTAL: 1
```

### 3. Parallel Builds

Use matrix builds to build multiple targets in parallel:

```yaml
strategy:
  matrix:
    target: [x86_64-unknown-linux-gnu, aarch64-unknown-linux-gnu]
```

## Advanced Usage

### Custom Build Flags

To add custom build flags, run `cargo`/`cross` directly instead:

```yaml
- name: Custom build
  run: |
    if [ "${{ matrix.target }}" = "aarch64-unknown-linux-gnu" ]; then
      cross build --release --target ${{ matrix.target }} --features my-feature
    else
      cargo build --release --target ${{ matrix.target }} --features my-feature
    fi
```

### Build Specific Binary

For workspace projects:

```yaml
- name: Build specific binary
  run: |
    cargo build --release --target ${{ matrix.target }} --bin my-specific-app
```

### Strip Debug Symbols

Reduce binary size:

```yaml
- name: Build and strip
  uses: firestoned/github-actions/rust/build-binary@v1
  with:
    target: ${{ matrix.target }}

- name: Strip binary
  run: |
    strip target/${{ matrix.target }}/release/my-app
```

## Related Actions

- [rust/setup-rust-build](../setup-rust-build/README.md) - Setup Rust environment (required)
- [rust/security-scan](../security-scan/README.md) - Security vulnerability scanning
- [rust/generate-sbom](../generate-sbom/README.md) - Generate Software Bill of Materials
- [docker/setup-docker](../../docker/setup-docker/README.md) - Setup Docker for cross-compilation

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Requires Docker for ARM64 builds
- **Rust versions**: Compatible with all Rust toolchain versions
- **Targets**: Currently optimized for x86_64 and aarch64 Linux

## Supported Targets

### Currently Implemented
- `x86_64-unknown-linux-gnu` - Native build with `cargo`
- `aarch64-unknown-linux-gnu` - Cross-build with `cross`

### Extensible for Additional Targets

To add support for more targets, modify the action to include additional conditions:

```yaml
- name: Build (release) - musl
  if: inputs.target == 'x86_64-unknown-linux-musl'
  shell: bash
  run: cargo build --release --target ${{ inputs.target }} --verbose
```

## Examples

### Complete Build and Release Workflow

```yaml
name: Build and Release

on:
  push:
    tags:
      - 'v*'

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

      - name: Build binary
        uses: firestoned/github-actions/rust/build-binary@v1
        with:
          target: ${{ matrix.target }}

      - name: Package binary
        run: |
          cd target/${{ matrix.target }}/release
          tar czf my-app-${{ matrix.target }}.tar.gz my-app

      - name: Upload to release
        uses: softprops/action-gh-release@v1
        with:
          files: target/${{ matrix.target }}/release/my-app-${{ matrix.target }}.tar.gz
```

### Build with Tests

```yaml
- name: Setup Rust
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: ${{ matrix.target }}

- name: Run tests
  run: cargo test --target ${{ matrix.target }}

- name: Build binary
  uses: firestoned/github-actions/rust/build-binary@v1
  with:
    target: ${{ matrix.target }}
```
