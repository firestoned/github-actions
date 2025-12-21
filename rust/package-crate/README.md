# Package Rust Crate

Composite action to package a Rust crate for publishing to crates.io. Supports both workspace and standalone crates.

## Features

- ✅ **Workspace Support**: Handles workspace crates with `version.workspace = true`
- ✅ **Standalone Support**: Works with single-crate projects
- ✅ **Allow Dirty**: Optional flag to package with uncommitted changes
- ✅ **Flexible Arguments**: Pass additional cargo arguments
- ✅ **Verification Output**: Lists packaged `.crate` files after successful packaging

## Usage

### Workspace Crate

For crates that use workspace version inheritance (`version.workspace = true`):

```yaml
- name: Package workspace crate
  uses: firestoned/github-actions/rust/package-crate@v1.2.4
  with:
    package: my-crate-name
    workspace: true
    allow-dirty: true
```

### Standalone Crate

For standalone crates (single Cargo.toml):

```yaml
- name: Package crate
  uses: firestoned/github-actions/rust/package-crate@v1.2.4
  with:
    allow-dirty: false
```

### With Additional Arguments

```yaml
- name: Package crate with features
  uses: firestoned/github-actions/rust/package-crate@v1.2.4
  with:
    package: my-crate
    workspace: true
    cargo-args: '--no-verify'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `package` | Package name to package (for workspace crates) | No | `''` |
| `workspace` | Whether this is a workspace crate (uses --package flag) | No | `false` |
| `allow-dirty` | Allow packaging with uncommitted changes | No | `true` |
| `cargo-args` | Additional arguments to pass to cargo package | No | `''` |

## Outputs

This action produces a `.crate` file in `target/package/` directory.

## Prerequisites

- Rust toolchain must be installed (use `firestoned/github-actions/rust/setup-rust-build`)
- For workspace crates, must be run from workspace root

## Example: Complete Release Workflow

```yaml
jobs:
  package-crates:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        crate:
          - name: my-derive-crate
          - name: my-runtime-crate
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1.2.4
        with:
          target: x86_64-unknown-linux-gnu

      - name: Update workspace version
        run: |
          sed -i 's/^version = ".*"/version = "1.0.0"/' Cargo.toml

      - name: Package crate
        uses: firestoned/github-actions/rust/package-crate@v1.2.4
        with:
          package: ${{ matrix.crate.name }}
          workspace: true

      - name: Upload package
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.crate.name }}-package
          path: target/package/${{ matrix.crate.name }}-*.crate
```

## Why Use This Action?

### Problem: Workspace Version Management

When using workspace version inheritance:

```toml
# Cargo.toml (workspace root)
[workspace.package]
version = "1.0.0"

# my-crate/Cargo.toml
[package]
version.workspace = true
```

Running `cd my-crate && cargo package` fails because the workspace version is not accessible from the crate directory.

### Solution: Package from Workspace Root

This action runs `cargo package --package <name>` from the workspace root, allowing cargo to properly resolve workspace-inherited fields.

## Notes

- The action uses `--allow-dirty` by default since CI often modifies `Cargo.toml` to update versions
- For workspace crates, `workspace: true` and `package: <name>` must both be specified
- The packaged `.crate` file is placed in `target/package/`

## License

MIT
