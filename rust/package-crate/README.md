# Package Rust Crate

Composite GitHub Action to package a Rust crate into a `.crate` archive for publishing to crates.io. Supports both standalone crates and workspace crates with inherited versions.

Packaging a crate before publishing lets you inspect what will be uploaded, catch missing files, and archive the exact artifact that will land on crates.io. This action wraps `cargo package` with workspace-aware logic and produces reusable outputs for downstream steps.

## Features

- **Workspace Support**: Handles workspace crates with `version.workspace = true` — packages from the workspace root so cargo can resolve inherited fields
- **Standalone Support**: Works with single-crate repositories with no extra configuration
- **Output Paths**: Exposes `crate-path` and `crate-name` outputs for use in upload or publish steps
- **Allow Dirty**: Optional flag to package with uncommitted changes (useful when CI modifies `Cargo.toml` for version bumps)
- **Custom Arguments**: Pass any additional `cargo package` flags via `cargo-args`
- **Step Summary**: Writes a formatted summary table to `$GITHUB_STEP_SUMMARY`

## Usage

### Basic — Standalone Crate

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup Rust
    uses: firestoned/github-actions/rust/setup-rust-build@v1
    with:
      target: x86_64-unknown-linux-gnu

  - name: Package crate
    id: package
    uses: firestoned/github-actions/rust/package-crate@v1

  - name: Upload crate artifact
    uses: actions/upload-artifact@v4
    with:
      name: crate
      path: ${{ steps.package.outputs.crate-path }}
```

### Workspace Crate

For crates using workspace version inheritance (`version.workspace = true`):

```yaml
- name: Package workspace crate
  id: package
  uses: firestoned/github-actions/rust/package-crate@v1
  with:
    package: my-crate-name
    workspace: true
    allow-dirty: true
```

### With Additional Arguments

```yaml
- name: Package with no-verify
  uses: firestoned/github-actions/rust/package-crate@v1
  with:
    package: my-crate
    workspace: true
    cargo-args: '--no-verify'
```

### Using the Crate Path Output

```yaml
- name: Package crate
  id: package
  uses: firestoned/github-actions/rust/package-crate@v1
  with:
    package: my-crate
    workspace: true

- name: Inspect crate contents
  run: |
    tar -tvzf "${{ steps.package.outputs.crate-path }}"

- name: Upload to release
  uses: actions/upload-release-asset@v1
  with:
    asset_path: ${{ steps.package.outputs.crate-path }}
    asset_name: ${{ steps.package.outputs.crate-name }}
    asset_content_type: application/gzip
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `package` | Package name to package. Required when `workspace: true` | No | `''` |
| `workspace` | Set `true` when packaging a workspace member (enables `--package` flag) | No | `false` |
| `allow-dirty` | Allow packaging with uncommitted changes | No | `true` |
| `cargo-args` | Additional arguments passed verbatim to `cargo package` | No | `''` |

## Outputs

| Output | Description |
|--------|-------------|
| `crate-path` | Relative path to the first `.crate` file found in `target/package/` |
| `crate-name` | Filename of the packaged `.crate` file |

## Prerequisites

- Rust toolchain must be installed before calling this action. Use [`rust/setup-rust-build`](../setup-rust-build/README.md) or [`dtolnay/rust-toolchain`](https://github.com/dtolnay/rust-toolchain).
- For workspace crates, the action must run from the workspace root directory.
- `Cargo.toml` must be valid and all dependencies resolvable.

## Examples

### Complete Release Workflow — Single Crate

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  package-and-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: x86_64-unknown-linux-gnu

      - name: Package crate
        id: package
        uses: firestoned/github-actions/rust/package-crate@v1

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: crate-package
          path: ${{ steps.package.outputs.crate-path }}

      - name: Publish to crates.io
        uses: firestoned/github-actions/rust/publish-crate@v1
        with:
          token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Complete Release Workflow — Workspace with Multiple Crates

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  package-crates:
    runs-on: ubuntu-latest
    strategy:
      max-parallel: 1
      matrix:
        crate:
          - name: my-derive-crate
          - name: my-runtime-crate
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: x86_64-unknown-linux-gnu

      - name: Update workspace version to release tag
        run: |
          version="${GITHUB_REF_NAME#v}"
          sed -i "s/^version = \".*\"/version = \"${version}\"/" Cargo.toml

      - name: Package crate
        id: package
        uses: firestoned/github-actions/rust/package-crate@v1
        with:
          package: ${{ matrix.crate.name }}
          workspace: true
          allow-dirty: true

      - name: Upload crate artifact
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.crate.name }}-package
          path: ${{ steps.package.outputs.crate-path }}
```

### Dry Run Validation in PR

```yaml
name: Validate Package

on:
  pull_request:

jobs:
  validate-package:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: x86_64-unknown-linux-gnu

      - name: Verify crate packages cleanly
        uses: firestoned/github-actions/rust/package-crate@v1
        with:
          cargo-args: '--no-verify'
```

### Package and Sign with Cosign

```yaml
- name: Package crate
  id: package
  uses: firestoned/github-actions/rust/package-crate@v1
  with:
    package: my-crate
    workspace: true

- name: Sign crate artifact
  uses: firestoned/github-actions/security/cosign-sign@v1
  with:
    artifact-path: ${{ steps.package.outputs.crate-path }}
    registry: ghcr.io
    repository: ${{ github.repository }}
```

## How It Works

1. **Toolchain Verification**: Calls [`rust/verify-toolchain`](../verify-toolchain/README.md) to ensure `cargo` is present.
2. **Command Construction**: Builds the `cargo package` command from inputs — optionally adding `--package`, `--allow-dirty`, and any `cargo-args`.
3. **Workspace Handling**: When `workspace: true` is set, adds `--package <name>` so cargo resolves the crate from the workspace root rather than failing in the crate subdirectory.
4. **Output Capture**: Searches `target/package/` for the generated `.crate` file and writes paths to `$GITHUB_OUTPUT`.
5. **Step Summary**: Appends a summary table to `$GITHUB_STEP_SUMMARY` for visibility in the GitHub Actions UI.

## Why Package from the Workspace Root?

When using workspace version inheritance:

```toml
# Cargo.toml (workspace root)
[workspace.package]
version = "1.2.0"

# my-crate/Cargo.toml
[package]
version.workspace = true
```

Running `cd my-crate && cargo package` fails because cargo cannot resolve `version.workspace` outside the workspace context. This action always runs from the workspace root using `--package <name>`, which avoids this error entirely.

## Best Practices

1. **Commit `Cargo.lock`** — Ensures reproducible builds and deterministic `.crate` archives.
2. **Use `allow-dirty: true` in release workflows** — CI often modifies `Cargo.toml` to update the version number before packaging.
3. **Use `cargo-args: '--no-verify'` for quick validation** — Skips running tests during packaging; useful for PR checks.
4. **Upload the artifact** — Store the `.crate` file as a workflow artifact alongside your release binaries for traceability.
5. **Package before publish** — Running this action before `rust/publish-crate` lets you inspect what will be uploaded.

## Troubleshooting

### `error: failed to verify package tarball`

Cargo runs `cargo test` on the packaged crate by default to verify it works. If your project requires specific environment variables or build flags, use `cargo-args: '--no-verify'` to skip this step.

### `error: no matching package named '...'`

Ensure `workspace: true` is set and `package` matches the `name` field in the crate's `Cargo.toml`, not the directory name. Also confirm the action is running from the workspace root (`working-directory` is not set to a subdirectory).

### `No .crate files found in target/package`

This means `cargo package` exited successfully but produced no archive. Check that:
- The crate's `Cargo.toml` has a valid `[package]` section with `name` and `version`
- The crate is not excluded from packaging via `.gitignore` or `exclude` in `Cargo.toml`

### Cache Miss Slowing Down Packaging

Add [`rust/cache-cargo`](../cache-cargo/README.md) before this step to cache the Cargo registry and build artifacts.

## Advanced Usage

### Conditional Packaging Based on Changed Files

```yaml
- name: Detect changed crates
  id: changes
  uses: dorny/paths-filter@v3
  with:
    filters: |
      my-crate:
        - 'my-crate/**'

- name: Package my-crate
  if: steps.changes.outputs.my-crate == 'true'
  uses: firestoned/github-actions/rust/package-crate@v1
  with:
    package: my-crate
    workspace: true
```

### Matrix Strategy for Multiple Crates

```yaml
strategy:
  matrix:
    crate: [derive-crate, runtime-crate, cli-crate]
steps:
  - name: Package ${{ matrix.crate }}
    uses: firestoned/github-actions/rust/package-crate@v1
    with:
      package: ${{ matrix.crate }}
      workspace: true
```

## Compatibility

| Component | Requirement |
|-----------|-------------|
| Rust | stable, beta, or nightly |
| Cargo | Any version supporting `--package` |
| OS | Linux, macOS, Windows |
| Runners | ubuntu-latest, macos-latest, windows-latest, self-hosted |
| Workspace | Supported (use `workspace: true`) |

## Related Actions

- [`rust/publish-crate`](../publish-crate/README.md) — Publish a packaged crate to crates.io
- [`rust/setup-rust-build`](../setup-rust-build/README.md) — Set up Rust toolchain before packaging
- [`rust/security-scan`](../security-scan/README.md) — Scan Cargo dependencies for vulnerabilities
- [`security/cosign-sign`](../../security/cosign-sign/README.md) — Sign the packaged artifact with Cosign

## License

MIT License — see [LICENSE](../../LICENSE) for details.

## Contributing

Contributions welcome! Please see [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.
