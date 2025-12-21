# Verify Rust Toolchain

A composite GitHub Action that verifies the Rust toolchain and required components are installed before running Rust-related tasks.

## Purpose

This action provides a reusable way to verify that the Rust toolchain and specific components (rustfmt, clippy, llvm-tools-preview) are available in the workflow environment. It's used internally by other Rust actions in this repository to ensure prerequisites are met.

## Features

- **Flexible Component Verification** - Check for cargo, rustfmt, clippy, or llvm-tools-preview
- **Clear Error Messages** - Provides actionable guidance when components are missing
- **Version Outputs** - Returns version information for all verified tools
- **Composable** - Used by other actions to ensure prerequisites are met
- **Fail-Fast** - Exits immediately with clear errors if required components are missing

## Usage

### Basic Example (Verify Cargo Only)

```yaml
- name: Verify Rust toolchain
  uses: firestoned/github-actions/rust/verify-toolchain@v1
```

This verifies that cargo is available (the default behavior).

### Verify Multiple Components

```yaml
- name: Verify Rust with rustfmt and clippy
  uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-rustfmt: true
    require-clippy: true
```

### Verify All Common Components

```yaml
- name: Verify complete Rust toolchain
  uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: true
    require-rustfmt: true
    require-clippy: true
    require-llvm-tools: true
```

### Skip Cargo Verification (Components Only)

```yaml
- name: Verify only rustfmt
  uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: false
    require-rustfmt: true
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `require-cargo` | Require cargo to be available | No | `true` |
| `require-rustfmt` | Require rustfmt component | No | `false` |
| `require-clippy` | Require clippy component | No | `false` |
| `require-llvm-tools` | Require llvm-tools-preview component | No | `false` |

## Outputs

| Name | Description |
|------|-------------|
| `rustc-version` | Rust compiler version (if cargo is available) |
| `cargo-version` | Cargo version (if cargo is available) |
| `rustfmt-version` | rustfmt version (if rustfmt is available) |
| `clippy-version` | clippy version (if clippy is available) |

## How It Works

1. **Cargo Verification** (if `require-cargo: true`)
   - Checks if `cargo` command is available
   - Outputs rustc and cargo versions
   - Provides setup instructions if missing

2. **rustfmt Verification** (if `require-rustfmt: true`)
   - Checks if `rustfmt` command is available
   - Outputs rustfmt version
   - Provides installation instructions if missing

3. **clippy Verification** (if `require-clippy: true`)
   - Checks if `cargo-clippy` command is available
   - Outputs clippy version
   - Provides installation instructions if missing

4. **llvm-tools Verification** (if `require-llvm-tools: true`)
   - Checks if llvm-tools-preview component is installed
   - Confirms installation
   - Provides installation instructions if missing

5. **Exit Handling**
   - Exits with error code 1 if any required component is missing
   - Provides clear, actionable error messages
   - References our setup-rust-build action for easy fixes

## Examples

### In a Linting Workflow

```yaml
name: Lint Rust Code

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          components: rustfmt, clippy

      - name: Verify toolchain
        uses: firestoned/github-actions/rust/verify-toolchain@v1
        with:
          require-rustfmt: true
          require-clippy: true

      - name: Run linters
        run: |
          cargo fmt --check
          cargo clippy -- -D warnings
```

### In a Build Workflow

```yaml
name: Build Rust Binary

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1

      - name: Verify toolchain
        uses: firestoned/github-actions/rust/verify-toolchain@v1

      - name: Build
        run: cargo build --release
```

### With Version Outputs

```yaml
- name: Verify Rust toolchain
  id: verify
  uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: true
    require-clippy: true

- name: Display versions
  run: |
    echo "Rust version: ${{ steps.verify.outputs.rustc-version }}"
    echo "Cargo version: ${{ steps.verify.outputs.cargo-version }}"
    echo "Clippy version: ${{ steps.verify.outputs.clippy-version }}"
```

### In Coverage Workflows

```yaml
- name: Verify Rust with llvm-tools
  uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: true
    require-llvm-tools: true

- name: Generate coverage
  run: |
    cargo install cargo-llvm-cov
    cargo llvm-cov --all-features --workspace
```

## Error Messages

When a required component is missing, the action provides clear, actionable error messages:

### Missing Cargo

```
❌ Error: Rust toolchain not found

Please set up Rust before using this action:

  - uses: firestoned/github-actions/rust/setup-rust-build@v1

Or use the community action:
  - uses: actions-rust-lang/setup-rust-toolchain@v1
```

### Missing rustfmt

```
❌ Error: rustfmt component not found

Install rustfmt:
  rustup component add rustfmt

Or use our setup action with rustfmt included:
  - uses: firestoned/github-actions/rust/setup-rust-build@v1
    with:
      components: rustfmt
```

### Missing clippy

```
❌ Error: clippy component not found

Install clippy:
  rustup component add clippy

Or use our setup action with clippy included:
  - uses: firestoned/github-actions/rust/setup-rust-build@v1
    with:
      components: clippy
```

## Used By

This action is used internally by other Rust actions in this repository:

- [**rust/lint**](../lint/README.md) - Requires rustfmt and clippy
- [**rust/build-binary**](../build-binary/README.md) - Requires cargo
- [**rust/build-library**](../build-library/README.md) - Requires cargo
- [**rust/generate-sbom**](../generate-sbom/README.md) - Requires cargo

## Best Practices

### 1. Use After Setup Action

Always run verify-toolchain after setting up the Rust toolchain:

```yaml
- uses: firestoned/github-actions/rust/setup-rust-build@v1

- uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-rustfmt: true
```

### 2. Only Verify What You Need

Don't require components you won't use:

```yaml
# ❌ Over-verification
- uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: true
    require-rustfmt: true
    require-clippy: true
    require-llvm-tools: true  # Not needed for this workflow

# ✅ Verify only what's needed
- uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: true
    require-rustfmt: true
```

### 3. Use Outputs for Debugging

Capture version outputs for debugging and audit trails:

```yaml
- name: Verify toolchain
  id: verify
  uses: firestoned/github-actions/rust/verify-toolchain@v1

- name: Log versions
  run: |
    echo "::notice::Rust version: ${{ steps.verify.outputs.rustc-version }}"
```

### 4. Fail Fast

Put verification early in your workflow to fail fast if prerequisites are missing:

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: firestoned/github-actions/rust/setup-rust-build@v1

  - uses: firestoned/github-actions/rust/verify-toolchain@v1  # Verify early
    with:
      require-cargo: true

  - name: Build (only runs if verification passes)
    run: cargo build
```

## Troubleshooting

### Verification Fails After Setup

**Problem**: Verification fails even after running setup-rust-build

**Solution**: Ensure components are specified in setup-rust-build:
```yaml
- uses: firestoned/github-actions/rust/setup-rust-build@v1
  with:
    components: rustfmt, clippy  # Explicitly include required components

- uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-rustfmt: true
    require-clippy: true
```

### Component Installed But Not Found

**Problem**: Component is installed but verification fails

**Solution**: Ensure the correct command is available:
- rustfmt requires `rustfmt` command
- clippy requires `cargo-clippy` command
- llvm-tools requires component to be in `rustup component list --installed`

Check manually:
```bash
rustup component list --installed
which rustfmt
which cargo-clippy
```

### Version Output is Empty

**Problem**: Version outputs are empty

**Solution**: Outputs are only populated if the component is required and available:
```yaml
- uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: true  # Must be true to get cargo-version output
```

## Advanced Usage

### Custom Verification Logic

For custom verification needs, use this action as a template and extend it:

```yaml
- name: Verify toolchain
  uses: firestoned/github-actions/rust/verify-toolchain@v1
  with:
    require-cargo: true

- name: Additional custom verification
  shell: bash
  run: |
    # Verify specific cargo plugins
    if ! cargo --list | grep -q "audit"; then
      echo "Installing cargo-audit..."
      cargo install cargo-audit
    fi
```

### Matrix Testing with Different Components

Test with different component combinations:

```yaml
strategy:
  matrix:
    components:
      - name: 'basic'
        rustfmt: false
        clippy: false
      - name: 'linting'
        rustfmt: true
        clippy: true
      - name: 'coverage'
        rustfmt: false
        clippy: false
        llvm-tools: true

steps:
  - uses: firestoned/github-actions/rust/verify-toolchain@v1
    with:
      require-rustfmt: ${{ matrix.components.rustfmt }}
      require-clippy: ${{ matrix.components.clippy }}
      require-llvm-tools: ${{ matrix.components.llvm-tools || false }}
```

## Related Actions

- [**rust/setup-rust-build**](../setup-rust-build/README.md) - Set up Rust toolchain with components
- [**rust/lint**](../lint/README.md) - Lint Rust code (uses this action internally)
- [**rust/build-binary**](../build-binary/README.md) - Build Rust binaries (uses this action internally)
- [**rust/build-library**](../build-library/README.md) - Build Rust libraries (uses this action internally)

## Compatibility

- **GitHub-hosted runners**: ✅ ubuntu-latest, macos-latest, windows-latest
- **Self-hosted runners**: ✅ Supported (requires Rust toolchain to be pre-installed or installed via setup action)
- **Rust versions**: Any stable, beta, or nightly with requested components available

## License

This action is licensed under the MIT License - see the [LICENSE](../../LICENSE) file for details.

---

**Author**: Erick Bourgeois
**Organization**: firestoned
**Repository**: [firestoned/github-actions](https://github.com/firestoned/github-actions)
