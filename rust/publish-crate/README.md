# Publish Rust Crate

Composite action to publish a Rust crate to crates.io. Supports both workspace and standalone crates.

## Features

- ✅ **Workspace Support**: Handles workspace crates with `version.workspace = true`
- ✅ **Standalone Support**: Works with single-crate projects
- ✅ **Modern Authentication**: Uses `cargo login` with environment variables (recommended approach)
- ✅ **Secure Token Handling**: Token passed via environment variable, never exposed in command line
- ✅ **Allow Dirty**: Optional flag to publish with uncommitted changes
- ✅ **Dry Run**: Test publishing without actually uploading
- ✅ **Flexible Arguments**: Pass additional cargo arguments

## Usage

### Workspace Crate

For crates that use workspace version inheritance (`version.workspace = true`):

```yaml
- name: Publish workspace crate
  uses: firestoned/github-actions/rust/publish-crate@v1.2.4
  with:
    package: my-crate-name
    workspace: true
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Standalone Crate

For standalone crates (single Cargo.toml):

```yaml
- name: Publish crate
  uses: firestoned/github-actions/rust/publish-crate@v1.2.4
  with:
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
    allow-dirty: false
```

### Dry Run

Test publishing without actually uploading to crates.io:

```yaml
- name: Test publish (dry run)
  uses: firestoned/github-actions/rust/publish-crate@v1.2.4
  with:
    package: my-crate
    workspace: true
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
    dry-run: true
```

### With Additional Arguments

```yaml
- name: Publish with custom registry
  uses: firestoned/github-actions/rust/publish-crate@v1.2.4
  with:
    package: my-crate
    workspace: true
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
    cargo-args: '--registry my-registry'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `package` | Package name to publish (for workspace crates) | No | `''` |
| `workspace` | Whether this is a workspace crate (uses --package flag) | No | `false` |
| `token` | crates.io API token (from secrets.CARGO_REGISTRY_TOKEN) | **Yes** | - |
| `allow-dirty` | Allow publishing with uncommitted changes | No | `true` |
| `dry-run` | Perform a dry run without actually publishing | No | `false` |
| `cargo-args` | Additional arguments to pass to cargo publish | No | `''` |

## Outputs

None. The crate is published directly to crates.io.

## Prerequisites

- Rust toolchain must be installed (use `firestoned/github-actions/rust/setup-rust-build`)
- For workspace crates, must be run from workspace root
- Valid crates.io API token stored in repository secrets

## Example: Complete Release Workflow

```yaml
jobs:
  publish-crates:
    runs-on: ubuntu-latest
    needs: sign-artifacts
    strategy:
      max-parallel: 1  # Publish sequentially for dependencies
      matrix:
        crate:
          - name: my-derive-crate  # Publish derive crate first
          - name: my-runtime-crate # Then publish crate that depends on it
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1.2.4
        with:
          target: x86_64-unknown-linux-gnu

      - name: Update workspace version
        run: |
          sed -i 's/^version = ".*"/version = "1.0.0"/' Cargo.toml

      - name: Publish to crates.io
        uses: firestoned/github-actions/rust/publish-crate@v1.2.4
        with:
          package: ${{ matrix.crate.name }}
          workspace: true
          token: ${{ secrets.CARGO_REGISTRY_TOKEN }}

      - name: Wait for crate propagation
        if: matrix.crate.name == 'my-derive-crate'
        run: |
          echo "Waiting 60 seconds for crate to be indexed..."
          sleep 60
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

Running `cd my-crate && cargo publish` fails because the workspace version is not accessible from the crate directory.

### Solution: Publish from Workspace Root

This action runs `cargo publish --package <name>` from the workspace root, allowing cargo to properly resolve workspace-inherited fields.

## Security

- The action uses `cargo login` with the token passed via `CARGO_REGISTRY_TOKEN` environment variable
- The `token` input is masked in GitHub Actions logs
- Token is never exposed in command-line arguments
- Store your crates.io token in repository secrets as `CARGO_REGISTRY_TOKEN`

## How It Works

1. **Authentication**: Runs `cargo login` with token from `CARGO_REGISTRY_TOKEN` environment variable
2. **Publishing**: Runs `cargo publish` (uses credentials from login step)
3. **Workspace Handling**: Uses `--package` flag when `workspace: true` is specified
4. **Dry Run**: Adds `--dry-run` flag when enabled for testing without publishing

## Publishing Order

For workspace crates with dependencies:
1. Use `max-parallel: 1` in your matrix strategy
2. Publish dependencies first (e.g., derive macros)
3. Add a wait step after dependency crates to allow crates.io indexing
4. Then publish dependent crates

## Notes

- The action uses `--allow-dirty` by default since CI often modifies `Cargo.toml` to update versions
- For workspace crates, `workspace: true` and `package: <name>` must both be specified
- Use `dry-run: true` to test the publishing process without uploading

## License

MIT
