# Lint Rust Code

A composite GitHub Action that runs `cargo fmt` and `cargo clippy` to check code formatting and quality for Rust projects.

## Features

- **Automated Formatting Checks** - Verify code follows Rust style guidelines with `cargo fmt`
- **Code Quality Analysis** - Catch common mistakes and improve code with `cargo clippy`
- **Flexible Configuration** - Enable/disable individual checks, customize clippy warnings
- **Feature Support** - Lint with specific features, all features, or no default features
- **Workspace Support** - Lint entire workspaces or specific packages
- **Fail-Fast Option** - Choose whether to fail on warnings or just report them
- **Detailed Summaries** - GitHub Actions step summaries with clear pass/fail indicators

## Usage

### Basic Example

```yaml
- name: Lint Rust code
  uses: firestoned/github-actions/rust/lint@v1
```

This runs both `cargo fmt --check` and `cargo clippy -- -D warnings` (fails on any warnings).

### Format Check Only

```yaml
- name: Check formatting
  uses: firestoned/github-actions/rust/lint@v1
  with:
    check-clippy: false
```

### Clippy Only

```yaml
- name: Run Clippy
  uses: firestoned/github-actions/rust/lint@v1
  with:
    check-fmt: false
```

### Custom Clippy Arguments

```yaml
- name: Lint with custom clippy settings
  uses: firestoned/github-actions/rust/lint@v1
  with:
    clippy-args: '-- -D warnings -W clippy::pedantic'
```

### Lint with All Features

```yaml
- name: Lint with all features
  uses: firestoned/github-actions/rust/lint@v1
  with:
    all-features: true
```

### Lint with Specific Features

```yaml
- name: Lint with specific features
  uses: firestoned/github-actions/rust/lint@v1
  with:
    features: 'serde,async'
```

### Lint Entire Workspace

```yaml
- name: Lint workspace
  uses: firestoned/github-actions/rust/lint@v1
  with:
    workspace: true
    all-features: true
```

### Lint Specific Package

```yaml
- name: Lint specific package
  uses: firestoned/github-actions/rust/lint@v1
  with:
    package: my-core-library
```

### Allow Warnings

```yaml
- name: Lint without failing on warnings
  uses: firestoned/github-actions/rust/lint@v1
  with:
    fail-on-warnings: false
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `check-fmt` | Run `cargo fmt --check` | No | `true` |
| `check-clippy` | Run `cargo clippy` | No | `true` |
| `clippy-args` | Additional arguments for cargo clippy | No | `-- -D warnings` |
| `all-features` | Check with all features enabled | No | `false` |
| `features` | Space or comma-separated list of features | No | `''` |
| `no-default-features` | Do not activate default features | No | `false` |
| `workspace` | Lint all workspace members | No | `false` |
| `package` | Package to lint (for workspaces) | No | `''` |
| `fail-on-warnings` | Fail if clippy emits warnings | No | `true` |

## Outputs

| Name | Description |
|------|-------------|
| `fmt-status` | Formatting check status: `success` or `failure` |
| `clippy-status` | Clippy check status: `success` or `failure` |

## How It Works

### Formatting Check (`cargo fmt`)

1. **Command**: `cargo fmt --all --check` (or with `--package` for specific packages)
2. **Behavior**: Checks if code is properly formatted without modifying files
3. **Failure**: Exits with error if any file needs formatting
4. **Fix**: Run `cargo fmt` locally to auto-format code

### Clippy Check (`cargo clippy`)

1. **Command**: `cargo clippy --all-targets [features] [clippy-args]`
2. **Behavior**: Analyzes code for common mistakes and style issues
3. **Default**: Fails on warnings (`-D warnings`)
4. **Customizable**: Pass custom lint levels via `clippy-args`

### Lint Levels

Clippy supports several lint levels:

| Flag | Description |
|------|-------------|
| `-D warnings` | Deny warnings (fail on any warning) |
| `-W warnings` | Warn on warnings (don't fail) |
| `-A warnings` | Allow warnings (suppress all) |
| `-D clippy::pedantic` | Deny pedantic lints |
| `-W clippy::nursery` | Warn on nursery lints (experimental) |

## Examples

### Complete CI Workflow

```yaml
name: Rust CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache dependencies
        uses: firestoned/github-actions/rust/cache-cargo@v1

      - name: Run linters
        uses: firestoned/github-actions/rust/lint@v1
        with:
          all-features: true

  build:
    name: Build
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        uses: firestoned/github-actions/rust/build-library@v1
```

### Feature Matrix Linting

```yaml
jobs:
  lint:
    strategy:
      matrix:
        features:
          - name: 'default'
            flags: ''
          - name: 'serde'
            flags: 'serde'
          - name: 'all'
            all: true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lint with ${{ matrix.features.name }} features
        uses: firestoned/github-actions/rust/lint@v1
        with:
          features: ${{ matrix.features.flags }}
          all-features: ${{ matrix.features.all || false }}
```

### Workspace Linting

```yaml
jobs:
  lint-workspace:
    name: Lint Entire Workspace
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lint workspace
        uses: firestoned/github-actions/rust/lint@v1
        with:
          workspace: true
          all-features: true

  lint-packages:
    name: Lint Individual Packages
    strategy:
      matrix:
        package:
          - my-core
          - my-cli
          - my-utils
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lint ${{ matrix.package }}
        uses: firestoned/github-actions/rust/lint@v1
        with:
          package: ${{ matrix.package }}
```

### Pedantic Clippy

```yaml
- name: Strict linting
  uses: firestoned/github-actions/rust/lint@v1
  with:
    clippy-args: '-- -D warnings -D clippy::pedantic -D clippy::cargo'
```

### Format-Only PR Check

```yaml
name: PR Format Check

on:
  pull_request:

jobs:
  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check formatting only
        uses: firestoned/github-actions/rust/lint@v1
        with:
          check-clippy: false
```

## Best Practices

### 1. Run Linting Early

Put linting before builds to fail fast:

```yaml
jobs:
  lint:
    # Runs first
  build:
    needs: lint  # Only runs if lint passes
```

### 2. Separate Format and Clippy

For better CI feedback, run as separate steps:

```yaml
- name: Check formatting
  uses: firestoned/github-actions/rust/lint@v1
  with:
    check-clippy: false

- name: Run Clippy
  uses: firestoned/github-actions/rust/lint@v1
  with:
    check-fmt: false
```

### 3. Use the Same Features

Lint with the same features you build with:

```yaml
- name: Lint
  uses: firestoned/github-actions/rust/lint@v1
  with:
    all-features: true

- name: Build
  uses: firestoned/github-actions/rust/build-library@v1
  with:
    all-features: true
```

### 4. Commit Formatting Locally

Set up a pre-commit hook to avoid CI failures:

```bash
# .git/hooks/pre-commit
#!/bin/sh
cargo fmt -- --check || exit 1
```

### 5. Use rustfmt.toml

Customize formatting rules in `rustfmt.toml`:

```toml
edition = "2021"
max_width = 100
use_small_heuristics = "Max"
```

### 6. Use clippy.toml

Configure clippy lints in `clippy.toml`:

```toml
# Warn on pedantic lints
pedantic-lints = "warn"

# Deny specific lints
avoid-breaking-exported-api = true
```

## Troubleshooting

### Formatting Check Fails

**Problem**: `cargo fmt --check` fails in CI but passes locally

**Solution**: Ensure same Rust version and rustfmt.toml:
```yaml
- uses: actions-rust-lang/setup-rust-toolchain@v1
  with:
    toolchain: stable
    components: rustfmt, clippy
```

### Clippy Warnings Not Failing

**Problem**: Warnings shown but action passes

**Solution**: Ensure `-D warnings` is in clippy-args:
```yaml
with:
  clippy-args: '-- -D warnings'
```

### Too Many Clippy Warnings

**Problem**: Legacy code has many clippy warnings

**Solution**: Start with warnings, gradually move to errors:
```yaml
# Phase 1: Just warn
with:
  clippy-args: '-- -W clippy::all'
  fail-on-warnings: false

# Phase 2: Deny after cleanup
with:
  clippy-args: '-- -D warnings'
```

### Workspace Package Not Found

**Problem**: `package` not found in workspace

**Solution**: Ensure package name matches exactly:
```yaml
with:
  package: my-package  # Must match [package] name in Cargo.toml
```

### Clippy Errors on Feature Combinations

**Problem**: Clippy fails with certain feature combinations

**Solution**: Test feature combinations explicitly:
```yaml
strategy:
  matrix:
    features: ['', 'serde', 'async', 'serde,async']
```

## Advanced Usage

### Auto-Fix Formatting

Use in a separate workflow to auto-fix formatting:

```yaml
name: Auto Format

on:
  pull_request:

jobs:
  format:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.head_ref }}

      - name: Run cargo fmt
        run: cargo fmt

      - name: Commit changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add -A
          git diff --quiet && git diff --staged --quiet || \
            git commit -m "chore: auto-format code [skip ci]"
          git push
```

### Custom Clippy Configuration per Package

For workspaces with different lint requirements:

```yaml
- name: Lint core (strict)
  uses: firestoned/github-actions/rust/lint@v1
  with:
    package: my-core
    clippy-args: '-- -D warnings -D clippy::pedantic'

- name: Lint CLI (relaxed)
  uses: firestoned/github-actions/rust/lint@v1
  with:
    package: my-cli
    clippy-args: '-- -D warnings'
```

### Clippy Fix Mode (Local Development)

Document for developers:

```bash
# Check for issues
cargo clippy -- -D warnings

# Auto-fix where possible
cargo clippy --fix

# Fix and allow dirty/staged files
cargo clippy --fix --allow-dirty --allow-staged
```

### GitHub Actions Annotations

Clippy automatically creates annotations in PR files. View them in:
- Files changed tab
- Annotations section of workflow run

## Related Actions

- [**rust/cache-cargo**](../cache-cargo/README.md) - Cache dependencies before linting
- [**rust/build-library**](../build-library/README.md) - Build after successful linting
- [**rust/security-scan**](../security-scan/README.md) - Security scanning
- [**security/license-check**](../../security/license-check/README.md) - License compliance

## Compatibility

- **GitHub-hosted runners**: ✅ ubuntu-latest, macos-latest, windows-latest
- **Self-hosted runners**: ✅ Supported (requires Rust toolchain with rustfmt and clippy)
- **Rust versions**: Any stable, beta, or nightly with rustfmt and clippy components

## License

This action is licensed under the MIT License - see the [LICENSE](../../LICENSE) file for details.

---

**Author**: Erick Bourgeois
**Organization**: firestoned
**Repository**: [firestoned/github-actions](https://github.com/firestoned/github-actions)
