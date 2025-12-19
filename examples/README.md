# Example Workflows

This directory contains example workflows demonstrating how to use the firestoned/github-actions composite actions in real-world scenarios.

## Available Examples

### [rust-library-ci.yml](rust-library-ci.yml)

**Purpose**: Comprehensive CI/CD pipeline for Rust library projects

**Demonstrates**:
- ✅ Feature matrix testing (default, minimal, specific features, all features)
- ✅ Multi-architecture builds (x86_64, ARM64)
- ✅ Code quality checks (formatting, clippy, documentation)
- ✅ Security scanning (cargo-audit, SBOM generation)
- ✅ Benchmark testing
- ✅ Documentation generation
- ✅ Workspace builds

**Actions Used**:
- `rust/cache-cargo` - Dependency caching
- `rust/setup-rust-build` - Toolchain setup with cross-compilation
- `rust/build-library` - Library building with profiles and features
- `rust/security-scan` - Vulnerability scanning
- `rust/generate-sbom` - Software Bill of Materials

**Key Features**:
- Matrix strategy for testing multiple feature combinations
- Conditional jobs (benchmarks and docs only on main branch)
- Artifact uploads for build outputs and SBOM
- CI success gate requiring all checks to pass

## How to Use These Examples

### 1. Copy to Your Repository

Copy the example workflow to your repository's `.github/workflows/` directory:

```bash
# From your repository root
mkdir -p .github/workflows
cp examples/rust-library-ci.yml .github/workflows/library-ci.yml
```

### 2. Customize for Your Project

Edit the workflow to match your project's needs:

**Update Feature Matrix**:
```yaml
matrix:
  features:
    - name: 'default'
      flags: ''
    - name: 'your-feature-name'
      flags: '--features your-feature'
```

**Update Targets** (if needed):
```yaml
matrix:
  target:
    - x86_64-unknown-linux-gnu
    - aarch64-unknown-linux-gnu
    - x86_64-apple-darwin  # Add macOS
```

**Enable/Disable Jobs**:
- Remove jobs you don't need (e.g., benchmarks, docs)
- Enable workspace job if you use Cargo workspaces

### 3. Configure Repository Settings

Ensure your repository has:

- **GitHub Actions enabled** (Settings → Actions → General)
- **Workflow permissions**: Read and write (for artifacts)
- **Branch protection** (optional): Require CI success before merge

## Example Adaptations

### Minimal Library CI

For a simpler CI pipeline, use just the essential jobs:

```yaml
jobs:
  test:
    # Build and test with default features

  quality:
    # Format and clippy checks

  security:
    # Security audit
```

### Feature-Rich Library CI

For comprehensive testing, include all jobs:

```yaml
jobs:
  test-features:    # Multiple feature combinations
  build-targets:    # Multi-architecture builds
  quality:          # Code quality
  security:         # Security scanning
  benchmark:        # Performance testing
  docs:             # Documentation generation
```

### Workspace Library CI

For Cargo workspaces with multiple crates:

```yaml
jobs:
  workspace:
    steps:
      - uses: firestoned/github-actions/rust/build-library@v1
        with:
          workspace: true
          all-features: true
```

## Testing Examples Locally

You can test these workflows locally using [act](https://github.com/nektos/act):

```bash
# Install act
brew install act  # macOS
# or
sudo apt install act  # Ubuntu

# Run the workflow
act -W examples/rust-library-ci.yml
```

## Common Customizations

### Add Custom Build Profiles

If you have custom Cargo profiles:

```yaml
# Cargo.toml
[profile.optimized]
inherits = "release"
lto = true
codegen-units = 1

# Workflow
- uses: firestoned/github-actions/rust/build-library@v1
  with:
    profile: optimized
```

### Test Against Multiple Rust Versions

```yaml
strategy:
  matrix:
    rust:
      - stable
      - beta
      - nightly
steps:
  - uses: actions-rust-lang/setup-rust-toolchain@v1
    with:
      toolchain: ${{ matrix.rust }}
```

### Add Code Coverage

```yaml
- name: Install tarpaulin
  run: cargo install cargo-tarpaulin

- name: Generate coverage
  run: cargo tarpaulin --all-features --out xml

- name: Upload to codecov
  uses: codecov/codecov-action@v4
```

## Best Practices

1. **Use Caching**: Always include `rust/cache-cargo` to speed up builds
2. **Matrix Testing**: Test critical feature combinations
3. **Security First**: Include security scanning in every workflow
4. **Fail Fast**: Set `fail-fast: false` to see all test results
5. **Conditional Jobs**: Run expensive jobs (benchmarks) only on main branch
6. **Artifacts**: Upload important build outputs and reports
7. **Documentation**: Generate and upload docs for releases

## Contributing Examples

Have a useful workflow pattern? Contribute it!

1. Create a new `.yml` file in this directory
2. Add comprehensive comments explaining the workflow
3. Update this README with a description
4. Submit a pull request

## Support

For questions about these examples:

- **Issues**: [GitHub Issues](https://github.com/firestoned/github-actions/issues)
- **Discussions**: [GitHub Discussions](https://github.com/firestoned/github-actions/discussions)
- **Documentation**: See individual action READMEs in the repository

## License

These examples are provided under the MIT License - see the [LICENSE](../LICENSE) file for details.
