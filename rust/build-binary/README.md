# Build Rust Binary

A composite GitHub Action that intelligently builds Rust binaries for specified target architectures, automatically using `cargo` for native builds and `cross` for cross-compilation.

## Features

- Automatic selection of build tool (`cargo` vs `cross`)
- Native builds for x86_64 Linux using standard `cargo`
- Native builds for Windows (MSVC and GNU) using standard `cargo`
- Cross-compilation for ARM64 Linux using `cross`
- Cross-compilation for ARM64 Windows using `cross`
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

### ARM64 Linux Cross-Compilation

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

### Windows x86_64 Build (MSVC - requires Windows runner)

```yaml
# Note: MSVC target requires Windows runners
runs-on: windows-latest

steps:
  - name: Setup Rust environment
    uses: firestoned/github-actions/rust/setup-rust-build@v1
    with:
      target: x86_64-pc-windows-msvc

  - name: Build binary
    uses: firestoned/github-actions/rust/build-binary@v1
    with:
      target: x86_64-pc-windows-msvc
```

### Windows x86_64 Build (GNU - works on Linux)

```yaml
# Works on Linux runners with mingw-w64
runs-on: ubuntu-latest

steps:
  - name: Install mingw-w64
    run: |
      sudo apt-get update
      sudo apt-get install -y mingw-w64

  - name: Setup Rust environment
    uses: firestoned/github-actions/rust/setup-rust-build@v1
    with:
      target: x86_64-pc-windows-gnu

  - name: Build binary
    uses: firestoned/github-actions/rust/build-binary@v1
    with:
      target: x86_64-pc-windows-gnu
```

### Multi-Architecture Matrix Build

```yaml
jobs:
  build:
    strategy:
      matrix:
        include:
          - target: x86_64-unknown-linux-gnu
            os: ubuntu-latest
          - target: aarch64-unknown-linux-gnu
            os: ubuntu-latest
          - target: x86_64-pc-windows-msvc
            os: ubuntu-latest
          - target: x86_64-pc-windows-gnu
            os: ubuntu-latest
          - target: aarch64-pc-windows-msvc
            os: ubuntu-latest
    runs-on: ${{ matrix.os }}
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
          path: target/${{ matrix.target }}/release/my-app${{ contains(matrix.target, 'windows') && '.exe' || '' }}
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `target` | Rust target triple (e.g., `x86_64-unknown-linux-gnu`, `x86_64-pc-windows-msvc`, `aarch64-unknown-linux-gnu`) | Yes | N/A |

## How It Works

### Decision Logic

The action automatically chooses the appropriate build tool based on the target:

| Target | Build Tool | Reason |
|--------|------------|--------|
| `x86_64-unknown-linux-gnu` | `cargo` | Native compilation on x86_64 runners |
| `aarch64-unknown-linux-gnu` | `cross` | Cross-compilation from x86_64 to ARM64 Linux |
| `x86_64-pc-windows-msvc` | `cargo` | Cross-compilation to Windows with MSVC toolchain |
| `x86_64-pc-windows-gnu` | `cargo` | Cross-compilation to Windows with GNU toolchain |
| `aarch64-pc-windows-msvc` | `cross` | Cross-compilation to ARM64 Windows |

### Build Process

#### For x86_64 Linux (Native Build)
```bash
cargo build --release --target x86_64-unknown-linux-gnu --verbose
```

#### For ARM64 Linux (Cross-Compilation)
```bash
cross build --release --target aarch64-unknown-linux-gnu --verbose
```

#### For x86_64 Windows MSVC (Cross-Compilation)
```bash
cargo build --release --target x86_64-pc-windows-msvc --verbose
```

#### For x86_64 Windows GNU (Cross-Compilation)
```bash
cargo build --release --target x86_64-pc-windows-gnu --verbose
```

#### For ARM64 Windows (Cross-Compilation)
```bash
cross build --release --target aarch64-pc-windows-msvc --verbose
```

All commands:
- Use `--release` for optimized production builds
- Use `--target` for explicit target specification
- Use `--verbose` for detailed build logs

## Output Location

Built binaries are located at:
```
target/{target}/release/{binary-name}[.exe]
```

For example:
- Linux: `target/x86_64-unknown-linux-gnu/release/my-app`
- Linux ARM64: `target/aarch64-unknown-linux-gnu/release/my-app`
- Windows: `target/x86_64-pc-windows-msvc/release/my-app.exe`
- Windows ARM64: `target/aarch64-pc-windows-msvc/release/my-app.exe`

**Note**: Windows binaries have a `.exe` extension

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

Use the standard Rust target directory structure, including `.exe` for Windows:

```yaml
- name: Upload binary
  uses: actions/upload-artifact@v4
  with:
    name: binary-${{ matrix.target }}
    path: target/${{ matrix.target }}/release/my-app${{ contains(matrix.target, 'windows') && '.exe' || '' }}
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
- Linux x86_64 servers: `x86_64-unknown-linux-gnu` or `x86_64-unknown-linux-musl`
- Linux ARM64 servers: `aarch64-unknown-linux-gnu`
- Windows x86_64: `x86_64-pc-windows-msvc` or `x86_64-pc-windows-gnu`
- Windows ARM64: `aarch64-pc-windows-msvc`
- macOS x86_64: `x86_64-apple-darwin`
- macOS ARM64: `aarch64-apple-darwin`

### Cross-Compilation Fails for ARM64 or Windows ARM64

**Problem**: `cross` build fails with Docker errors

**Solution**: Ensure Docker is available on the runner:
```yaml
- name: Setup Docker
  uses: docker/setup-buildx-action@v3
```

### Windows Build Fails with Missing Linker

**Problem**: Windows cross-compilation fails with "linker not found"

**Solution for GNU target**: Install mingw-w64:
```yaml
- name: Install Windows GNU dependencies
  if: matrix.target == 'x86_64-pc-windows-gnu'
  run: |
    sudo apt-get update
    sudo apt-get install -y mingw-w64
```

**Solution for MSVC target**: The MSVC target (`x86_64-pc-windows-msvc`) requires the Microsoft linker (`link.exe`) which is not available on Linux runners. You have two options:

1. **Use Windows runners** (recommended for MSVC):
```yaml
runs-on: windows-latest
```

2. **Use the GNU target instead** (works on Linux runners):
```yaml
target: x86_64-pc-windows-gnu
```

3. **Use xwin for MSVC on Linux** (advanced, requires additional setup):
```yaml
- name: Setup xwin for MSVC linker
  run: |
    cargo install xwin
    xwin --accept-license splat --output /tmp/xwin
    echo "XWIN_ARCH=x86_64" >> $GITHUB_ENV
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
    target:
      - x86_64-unknown-linux-gnu
      - aarch64-unknown-linux-gnu
      - x86_64-pc-windows-msvc
      - x86_64-pc-windows-gnu
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

**Linux Targets:**
- `x86_64-unknown-linux-gnu` - x86_64 Linux with GNU libc (native build with `cargo`)
- `aarch64-unknown-linux-gnu` - ARM64 Linux with GNU libc (cross-build with `cross`)

**Windows Targets:**
- `x86_64-pc-windows-msvc` - x86_64 Windows with MSVC toolchain (cross-build with `cargo`)
- `x86_64-pc-windows-gnu` - x86_64 Windows with GNU toolchain (cross-build with `cargo`)
- `aarch64-pc-windows-msvc` - ARM64 Windows with MSVC toolchain (cross-build with `cross`)

### Choosing Between Windows MSVC and GNU

**MSVC (`x86_64-pc-windows-msvc`)**:
- Uses Microsoft Visual C++ toolchain
- Better integration with Windows ecosystem
- Recommended for most Windows deployments
- **Requires Windows runners** (`runs-on: windows-latest`)
- Cannot cross-compile from Linux without complex setup (xwin)

**GNU (`x86_64-pc-windows-gnu`)** - Recommended for CI/CD:
- Uses MinGW-w64 toolchain
- **Works on Linux runners** with mingw-w64 installed
- Better for cross-platform CI pipelines
- Requires mingw-w64 installation: `sudo apt-get install mingw-w64`
- Good compatibility with most applications

**Recommendation for CI/CD**:
- Use `x86_64-pc-windows-gnu` on Linux runners (simpler, faster)
- Use `x86_64-pc-windows-msvc` on Windows runners (better Windows integration)
- Both produce fully functional Windows executables

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
        include:
          - target: x86_64-unknown-linux-gnu
            os: ubuntu-latest
            archive: tar.gz
          - target: aarch64-unknown-linux-gnu
            os: ubuntu-latest
            archive: tar.gz
          - target: x86_64-pc-windows-msvc
            os: ubuntu-latest
            archive: zip
          - target: aarch64-pc-windows-msvc
            os: ubuntu-latest
            archive: zip
    runs-on: ${{ matrix.os }}
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

      - name: Package binary (Linux)
        if: matrix.archive == 'tar.gz'
        run: |
          cd target/${{ matrix.target }}/release
          tar czf my-app-${{ matrix.target }}.tar.gz my-app

      - name: Package binary (Windows)
        if: matrix.archive == 'zip'
        run: |
          cd target/${{ matrix.target }}/release
          zip my-app-${{ matrix.target }}.zip my-app.exe

      - name: Upload to release
        uses: softprops/action-gh-release@v1
        with:
          files: target/${{ matrix.target }}/release/my-app-${{ matrix.target }}.${{ matrix.archive }}
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
