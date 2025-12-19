# Cache Cargo Dependencies

**Caches Cargo registry, index, and build artifacts for faster Rust builds**

This action caches three critical components of the Rust/Cargo build process:
1. **Cargo registry** (`~/.cargo/registry`) - Downloaded crate files
2. **Cargo index** (`~/.cargo/git`) - Git index for crates.io
3. **Build artifacts** (`target/`) - Compiled dependencies and incremental builds

## Features

- ✅ **Zero configuration** - Works out of the box for any Rust project
- ✅ **Multi-OS support** - Linux, macOS, Windows
- ✅ **Intelligent caching** - Uses `Cargo.lock` hash for cache keys
- ✅ **Incremental builds** - Caches build artifacts to speed up subsequent builds
- ✅ **Fallback keys** - Uses restore-keys for partial cache hits

## Usage

### Basic Usage

```yaml
- uses: actions/checkout@v4

- uses: firestoned/github-actions/rust/cache-cargo@v1

- name: Build
  run: cargo build --release
```

### Complete CI Workflow

```yaml
name: Rust CI

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

      # Cache Cargo dependencies
      - uses: firestoned/github-actions/rust/cache-cargo@v1

      # Install Rust toolchain
      - uses: dtolnay/rust-toolchain@stable

      # Build project
      - name: Build
        run: cargo build --release

      # Run tests
      - name: Test
        run: cargo test --all-features
```

### Multi-Platform Builds

```yaml
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4

      # Cache is OS-specific automatically
      - uses: firestoned/github-actions/rust/cache-cargo@v1

      - uses: dtolnay/rust-toolchain@stable

      - run: cargo build --release
```

## What Gets Cached

### 1. Cargo Registry (`~/.cargo/registry`)

Downloaded crate archives. This is the largest cache component and provides the most significant speed improvement.

**Cache key:** `${{ runner.os }}-cargo-registry-${{ hashFiles('**/Cargo.lock') }}`

### 2. Cargo Git Index (`~/.cargo/git`)

Git repositories for crates.io and other registries.

**Cache key:** `${{ runner.os }}-cargo-git-${{ hashFiles('**/Cargo.lock') }}`

### 3. Build Artifacts (`target/`)

Compiled dependencies and incremental compilation artifacts.

**Cache key:** `${{ runner.os }}-cargo-build-target-${{ hashFiles('**/Cargo.lock') }}`

## Cache Invalidation

The cache is automatically invalidated when:
- `Cargo.lock` changes (different hash)
- Operating system changes (cache keys include `runner.os`)
- Dependencies are added/removed/updated

## Performance Impact

**Typical build time improvements:**

| Scenario | Without Cache | With Cache | Improvement |
|----------|---------------|------------|-------------|
| Fresh build | ~5-10 min | ~5-10 min | 0% (first run) |
| Rebuild (no changes) | ~5-10 min | ~30-60 sec | **80-90%** |
| Rebuild (code changes) | ~5-10 min | ~1-2 min | **60-80%** |
| Rebuild (dependency changes) | ~5-10 min | ~2-4 min | **40-60%** |

## Advanced Usage

### Custom Cache Keys

If you need additional cache invalidation (e.g., for build configuration):

```yaml
- uses: firestoned/github-actions/rust/cache-cargo@v1

# Add your own cache for custom scenarios
- uses: actions/cache@v4
  with:
    path: target
    key: ${{ runner.os }}-custom-${{ hashFiles('build-config.toml') }}
```

### Workspace Projects

Works automatically with Cargo workspaces:

```yaml
# Caches all workspace members automatically
- uses: firestoned/github-actions/rust/cache-cargo@v1

- run: cargo build --workspace
```

### Cross-Compilation

Combine with multi-target builds:

```yaml
- uses: firestoned/github-actions/rust/cache-cargo@v1

- uses: dtolnay/rust-toolchain@stable
  with:
    targets: x86_64-unknown-linux-gnu,aarch64-unknown-linux-gnu

- run: cargo build --target x86_64-unknown-linux-gnu
- run: cargo build --target aarch64-unknown-linux-gnu
```

## Troubleshooting

### Cache Miss on Every Run

**Problem:** Cache is not being restored

**Solution:** Ensure `Cargo.lock` exists and is committed to the repository:
```bash
# Generate Cargo.lock if missing
cargo generate-lockfile

# Commit it
git add Cargo.lock
git commit -m "Add Cargo.lock for reproducible builds"
```

### Cache Size Limits

GitHub Actions has a 10GB cache limit per repository. If you hit this limit:

1. **Review cache usage:**
   ```bash
   gh cache list
   ```

2. **Clear old caches:**
   ```bash
   gh cache delete <cache-id>
   ```

3. **Consider excluding test artifacts:**
   - This action caches the entire `target/` directory
   - For very large projects, you may want to implement custom caching

### Slow Builds Despite Cache

**Problem:** Builds are still slow even with cache hits

**Possible causes:**
1. Large number of dependencies (cache restoration takes time)
2. Incremental compilation disabled
3. Build artifacts invalidated by environment changes

**Solution:**
```yaml
# Enable incremental compilation explicitly
- name: Build with incremental compilation
  env:
    CARGO_INCREMENTAL: 1
  run: cargo build --release
```

## Compatibility

- ✅ **Rust versions:** All versions (stable, beta, nightly)
- ✅ **Cargo versions:** All versions
- ✅ **Operating Systems:** Linux, macOS, Windows
- ✅ **GitHub Actions runners:** ubuntu-latest, macos-latest, windows-latest
- ✅ **Self-hosted runners:** Yes (with standard Cargo installation)

## Best Practices

1. **Always commit `Cargo.lock`** - Required for reproducible builds and effective caching
2. **Use specific Rust versions** - Avoid cache invalidation from toolchain updates
3. **Pin dependencies** - Use exact versions in `Cargo.toml` for stability
4. **Monitor cache size** - Large caches can slow down cache restoration
5. **Combine with Rust toolchain caching** - Use `Swatinem/rust-cache` for additional caching

## Related Actions

- [`rust/setup-rust-build`](../setup-rust-build/README.md) - Set up Rust toolchain with caching
- [`rust/build-binary`](../build-binary/README.md) - Build Rust binaries
- [`rust/security-scan`](../security-scan/README.md) - Scan dependencies for vulnerabilities

## License

MIT License - see [LICENSE](../../LICENSE) for details.

## Contributing

Contributions welcome! Please see [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.
