# Build Rust Library

A composite GitHub Action for building Rust libraries (crates) with flexible configuration for profiles, features, targets, and workspaces.

## Features

- **Flexible Build Profiles** - Support for `dev`, `release`, and custom profiles
- **Feature Management** - Activate specific features, all features, or disable defaults
- **Multi-Target Support** - Build for different architectures (x86_64, ARM64, etc.)
- **Workspace Support** - Build individual packages or entire workspaces
- **Auto Cross-Compilation** - Automatically uses `cross` for ARM64 builds
- **Library-Focused** - Optimized for library builds (skips bins, examples, tests)
- **Detailed Output** - Verbose logging and build summaries

## Usage

### Basic Library Build

```yaml
- name: Build library
  uses: firestoned/github-actions/rust/build-library@v1
```

This builds your library with the release profile using default features.

### Build with Specific Features

```yaml
- name: Build with features
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    features: serde,async
```

### Build with All Features

```yaml
- name: Build with all features
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    all-features: true
```

### Build for Specific Target (Cross-Compilation)

```yaml
- name: Setup Rust for ARM64
  uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-unknown-linux-gnu

- name: Build library for ARM64
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    target: aarch64-unknown-linux-gnu
```

### Build with Development Profile

```yaml
- name: Build with dev profile
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    profile: dev
```

### Build Specific Package in Workspace

```yaml
- name: Build specific package
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    package: my-core-library
```

### Build Entire Workspace

```yaml
- name: Build all workspace libraries
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    workspace: true
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `target` | Rust target triple (e.g., `x86_64-unknown-linux-gnu`, `aarch64-unknown-linux-gnu`) | No | `''` (host target) |
| `profile` | Build profile: `dev`, `release`, or custom profile name | No | `release` |
| `features` | Space or comma-separated list of features to activate | No | `''` |
| `all-features` | Activate all available features | No | `false` |
| `no-default-features` | Do not activate the default features | No | `false` |
| `lib` | Build only the library (skip bins, examples, tests, benches) | No | `true` |
| `package` | Package to build (for workspaces with multiple packages) | No | `''` |
| `workspace` | Build all workspace members | No | `false` |
| `use-cross` | Force use of cross for cross-compilation (`true`, `false`, `auto`) | No | `auto` |

## Outputs

| Name | Description |
|------|-------------|
| `build-status` | Build status: `success` or `failure` |
| `target-dir` | Path to target directory containing build artifacts |

## Examples

### Complete Library CI Workflow

```yaml
name: Library CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache Cargo dependencies
        uses: firestoned/github-actions/rust/cache-cargo@v1

      - name: Build library (release)
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          all-features: true

      - name: Run tests
        run: cargo test --all-features

      - name: Generate docs
        run: cargo doc --all-features --no-deps
```

### Multi-Target Build Matrix

```yaml
jobs:
  build:
    strategy:
      matrix:
        target:
          - x86_64-unknown-linux-gnu
          - aarch64-unknown-linux-gnu
          - x86_64-apple-darwin
          - aarch64-apple-darwin
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: ${{ matrix.target }}

      - name: Build library
        id: build
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          target: ${{ matrix.target }}

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: library-${{ matrix.target }}
          path: ${{ steps.build.outputs.target-dir }}/lib*
```

### Feature Matrix Testing

```yaml
jobs:
  test-features:
    strategy:
      matrix:
        features:
          - ''                    # Default features only
          - 'serde'              # Single feature
          - 'serde,async'        # Multiple features
          - 'all'                # All features
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build with features
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          features: ${{ matrix.features == 'all' && '' || matrix.features }}
          all-features: ${{ matrix.features == 'all' }}

      - name: Test with features
        run: |
          if [ "${{ matrix.features }}" = "all" ]; then
            cargo test --all-features
          elif [ -n "${{ matrix.features }}" ]; then
            cargo test --features "${{ matrix.features }}"
          else
            cargo test
          fi
```

### Workspace Build Example

```yaml
jobs:
  build-workspace:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache dependencies
        uses: firestoned/github-actions/rust/cache-cargo@v1

      # Build all workspace libraries
      - name: Build workspace
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          workspace: true
          all-features: true

      # Or build specific packages
      - name: Build core library
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          package: my-core
          features: serde

      - name: Build utils library
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          package: my-utils
          no-default-features: true
```

### Custom Build Profile

```yaml
# Cargo.toml
# [profile.bench]
# inherits = "release"
# lto = true
# codegen-units = 1

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build with custom profile
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          profile: bench
```

### No Default Features

```yaml
- name: Build minimal library
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    no-default-features: true
    features: minimal-core
```

## How It Works

### Build Command Construction

The action builds a `cargo build` or `cross build` command based on your inputs:

**Example 1**: Basic release build
```bash
cargo build --release --lib --verbose
```

**Example 2**: With features and target
```bash
cargo build --release --target aarch64-unknown-linux-gnu --lib --features serde,async --verbose
```

**Example 3**: All features, no defaults
```bash
cargo build --release --lib --all-features --no-default-features --verbose
```

**Example 4**: Workspace package with custom profile
```bash
cargo build --profile bench --package my-core --lib --verbose
```

### Cross-Compilation

The action automatically detects when to use `cross`:

| Scenario | Tool Used | Reason |
|----------|-----------|--------|
| `target: aarch64-unknown-linux-gnu` | `cross` | Cross-compilation to ARM64 |
| `use-cross: true` | `cross` | Explicitly requested |
| `use-cross: false` | `cargo` | Native build |
| No target specified | `cargo` | Host platform build |

### Target Directory

Build artifacts are placed in:
- **Release**: `target/release/` or `target/{target}/release/`
- **Dev**: `target/debug/` or `target/{target}/debug/`
- **Custom**: `target/{profile}/` or `target/{target}/{profile}/`

## Best Practices

### 1. Use Feature Flags for Optional Dependencies

```toml
# Cargo.toml
[features]
default = ["std"]
std = []
serde = ["dep:serde"]
async = ["tokio"]

[dependencies]
serde = { version = "1.0", optional = true }
tokio = { version = "1", optional = true }
```

```yaml
# Build with async feature
- uses: firestoned/github-actions/rust/build-library@v1
  with:
    features: async
```

### 2. Test Multiple Feature Combinations

Use matrix builds to ensure all feature combinations work:

```yaml
strategy:
  matrix:
    features:
      - ''
      - 'serde'
      - 'async'
      - 'serde,async'
```

### 3. Cache Dependencies

Always use caching before building:

```yaml
- uses: firestoned/github-actions/rust/cache-cargo@v1
- uses: firestoned/github-actions/rust/build-library@v1
```

### 4. Build for Multiple Targets

For cross-platform libraries, test on all supported targets:

```yaml
strategy:
  matrix:
    include:
      - target: x86_64-unknown-linux-gnu
        os: ubuntu-latest
      - target: aarch64-unknown-linux-gnu
        os: ubuntu-latest
      - target: x86_64-apple-darwin
        os: macos-latest
```

### 5. Use Outputs for Artifacts

Reference the output directory for uploading artifacts:

```yaml
- id: build
  uses: firestoned/github-actions/rust/build-library@v1

- uses: actions/upload-artifact@v4
  with:
    name: library
    path: ${{ steps.build.outputs.target-dir }}/lib*.a
```

### 6. Separate Build and Test

Build once, test with different configurations:

```yaml
- name: Build library
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    all-features: true

- name: Test default features
  run: cargo test

- name: Test all features
  run: cargo test --all-features

- name: Test no default features
  run: cargo test --no-default-features
```

## Troubleshooting

### Build Fails with "package not found"

**Problem**: Building a specific package in a workspace fails.

**Solution**: Ensure the package name matches exactly:
```yaml
with:
  package: my-package  # Must match [package] name in Cargo.toml
```

### Cross-compilation fails

**Problem**: ARM64 build fails even though `cross` is installed.

**Solution**: Ensure `setup-rust-build` was run first:
```yaml
- uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    target: aarch64-unknown-linux-gnu
```

### Features not activating

**Problem**: Features aren't being activated.

**Solution**: Check feature names match your `Cargo.toml`:
```toml
[features]
my-feature = []  # Use "my-feature" not "my_feature"
```

### Custom profile not found

**Problem**: Build fails with custom profile.

**Solution**: Ensure profile is defined in `Cargo.toml`:
```toml
[profile.my-custom-profile]
inherits = "release"
```

### Target directory is empty

**Problem**: Output directory doesn't contain expected artifacts.

**Solution**:
1. Check the build succeeded
2. Verify you're looking in the correct profile directory (`release` vs `debug`)
3. For libraries, look for `.rlib` or `.a` files, not binaries

## Comparison with build-binary

| Feature | build-library | build-binary |
|---------|--------------|--------------|
| **Purpose** | Build Rust libraries (crates) | Build Rust executables |
| **Default Flag** | `--lib` | None (builds bins by default) |
| **Feature Control** | Full feature flag support | Limited |
| **Workspace Support** | Yes | No |
| **Profile Support** | Full (dev, release, custom) | Release only |
| **Use Case** | Library development, crate publishing | Application builds, deployable binaries |

## Related Actions

- [**rust/cache-cargo**](../cache-cargo/README.md) - Cache Cargo dependencies
- [**rust/setup-rust-build**](../setup-rust-build/README.md) - Set up Rust toolchain
- [**rust/build-binary**](../build-binary/README.md) - Build Rust binaries
- [**rust/generate-sbom**](../generate-sbom/README.md) - Generate Software Bill of Materials

## Compatibility

- **GitHub-hosted runners**: ✅ ubuntu-latest, macos-latest, windows-latest
- **Self-hosted runners**: ✅ Supported (requires Rust toolchain)
- **Rust versions**: Any stable, beta, or nightly
- **Cargo versions**: 1.60+

## License

This action is licensed under the MIT License - see the [LICENSE](../../LICENSE) file for details.

---

**Author**: Erick Bourgeois
**Organization**: firestoned
**Repository**: [firestoned/github-actions](https://github.com/firestoned/github-actions)
