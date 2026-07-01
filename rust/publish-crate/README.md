# Publish Rust Crate

Composite GitHub Action to publish a Rust crate to crates.io. Supports both standalone crates and workspace crates with inherited versions. Handles authentication securely and provides a dry-run mode for validation without publishing.

## Features

- **Workspace Support**: Handles workspace crates with `version.workspace = true` — publishes from the workspace root so cargo can resolve inherited fields
- **Standalone Support**: Works with single-crate repositories out of the box
- **Secure Authentication**: Passes the crates.io token via stdin to `cargo login`, never exposing it on the command line or in shell history
- **Dry Run Mode**: Validate the publish process without uploading to crates.io
- **Status Output**: Exposes `publish-status` output (`published` or `dry-run`) for conditional downstream steps
- **Allow Dirty**: Optional flag for workflows that modify `Cargo.toml` before publishing
- **Custom Arguments**: Pass any additional `cargo publish` flags via `cargo-args`
- **Step Summary**: Writes a formatted summary to `$GITHUB_STEP_SUMMARY`

## Usage

### Basic — Standalone Crate

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup Rust
    uses: firestoned/github-actions/rust/setup-rust-build@v1
    with:
      target: x86_64-unknown-linux-gnu

  - name: Publish to crates.io
    uses: firestoned/github-actions/rust/publish-crate@v1
    with:
      token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Workspace Crate

For crates using workspace version inheritance (`version.workspace = true`):

```yaml
- name: Publish workspace crate
  uses: firestoned/github-actions/rust/publish-crate@v1
  with:
    package: my-crate-name
    workspace: true
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Dry Run Validation

Test the publish process without actually uploading:

```yaml
- name: Validate publish (dry run)
  uses: firestoned/github-actions/rust/publish-crate@v1
  with:
    package: my-crate
    workspace: true
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
    dry-run: true
```

### Using the Publish Status Output

```yaml
- name: Publish crate
  id: publish
  uses: firestoned/github-actions/rust/publish-crate@v1
  with:
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}

- name: Notify on successful publish
  if: steps.publish.outputs.publish-status == 'published'
  run: |
    echo "Published successfully to crates.io"
```

### With Custom Registry

```yaml
- name: Publish to custom registry
  uses: firestoned/github-actions/rust/publish-crate@v1
  with:
    package: my-crate
    workspace: true
    token: ${{ secrets.CUSTOM_REGISTRY_TOKEN }}
    cargo-args: '--registry my-registry'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `package` | Package name to publish. Required when `workspace: true` | No | `''` |
| `workspace` | Set `true` when publishing a workspace member (enables `--package` flag) | No | `false` |
| `token` | crates.io API token. Store as `CARGO_REGISTRY_TOKEN` in repository secrets | **Yes** | — |
| `allow-dirty` | Allow publishing with uncommitted changes | No | `true` |
| `dry-run` | Perform a dry run — validates without uploading | No | `false` |
| `cargo-args` | Additional arguments passed verbatim to `cargo publish` | No | `''` |

## Outputs

| Output | Description |
|--------|-------------|
| `publish-status` | `published` when successfully published, `dry-run` when dry-run mode was used |

## Prerequisites

- Rust toolchain must be installed before calling this action. Use [`rust/setup-rust-build`](../setup-rust-build/README.md).
- A valid crates.io API token stored in repository secrets as `CARGO_REGISTRY_TOKEN`.
- For workspace crates, the action must run from the workspace root directory.
- The crate's `Cargo.toml` must have `description`, `license`, and `repository` fields (crates.io requirement).

## Examples

### Complete Release Workflow — Single Crate

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: x86_64-unknown-linux-gnu

      - name: Publish to crates.io
        uses: firestoned/github-actions/rust/publish-crate@v1
        with:
          token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Complete Release Workflow — Workspace with Multiple Crates

When publishing multiple workspace crates where one depends on another, the dependency must be published and indexed by crates.io before the dependent crate can be published.

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  publish-crates:
    runs-on: ubuntu-latest
    needs: sign-artifacts
    strategy:
      max-parallel: 1  # Publish sequentially — dependencies first
      matrix:
        crate:
          - name: my-derive-crate   # Publish derive macros first
          - name: my-runtime-crate  # Then the crate that uses them
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

      - name: Publish to crates.io
        id: publish
        uses: firestoned/github-actions/rust/publish-crate@v1
        with:
          package: ${{ matrix.crate.name }}
          workspace: true
          token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
          allow-dirty: true

      - name: Wait for crates.io indexing
        if: matrix.crate.name == 'my-derive-crate'
        run: |
          echo "Waiting 60 seconds for crate to be indexed by crates.io..."
          sleep 60
```

### PR Validation with Dry Run

```yaml
name: Validate Release

on:
  pull_request:
    branches: [main]

jobs:
  validate-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: x86_64-unknown-linux-gnu

      - name: Dry run publish validation
        uses: firestoned/github-actions/rust/publish-crate@v1
        with:
          package: my-crate
          workspace: true
          token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
          dry-run: true
```

### Package Then Publish

Using both `package-crate` and `publish-crate` for maximum control:

```yaml
- name: Package crate
  id: package
  uses: firestoned/github-actions/rust/package-crate@v1
  with:
    package: my-crate
    workspace: true

- name: Inspect package contents
  run: tar -tvzf "${{ steps.package.outputs.crate-path }}"

- name: Publish to crates.io
  uses: firestoned/github-actions/rust/publish-crate@v1
  with:
    package: my-crate
    workspace: true
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

### Conditional Publish on Tag

```yaml
- name: Publish (release tags only)
  if: startsWith(github.ref, 'refs/tags/v')
  uses: firestoned/github-actions/rust/publish-crate@v1
  with:
    token: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

## How It Works

1. **Toolchain Verification**: Calls [`rust/verify-toolchain`](../verify-toolchain/README.md) to ensure `cargo` is present.
2. **Authentication**: Pipes the token from `$CARGO_REGISTRY_TOKEN` via stdin to `cargo login` — the token never appears in process arguments or shell history.
3. **Command Construction**: Builds the `cargo publish` command from inputs — optionally adding `--package`, `--allow-dirty`, `--dry-run`, and any `cargo-args`.
4. **Workspace Handling**: When `workspace: true`, adds `--package <name>` so cargo resolves the crate from the workspace root rather than failing in a subdirectory.
5. **Status Output**: Writes `publish-status` to `$GITHUB_OUTPUT` for use in downstream conditional steps.
6. **Step Summary**: Appends a summary table to `$GITHUB_STEP_SUMMARY`.

## Security

- The `token` input value is masked in GitHub Actions logs via the secrets mechanism
- The token is passed to `cargo login` via stdin (`echo ... | cargo login`), never on the command line
- Authentication credentials are not stored in environment variables visible to child processes after the login step

## Publishing Order for Workspace Crates

When a workspace contains crates with dependencies between them:

1. Use `max-parallel: 1` in your matrix strategy to enforce sequential publishing
2. Publish dependencies first (e.g., derive macro crates before runtime crates)
3. Add a `sleep 60` wait after each dependency to allow crates.io indexing before the next publish

Attempting to publish a crate before its dependency is indexed by crates.io will cause a `package not found` error.

## Best Practices

1. **Store tokens in repository secrets** — Never hard-code `CARGO_REGISTRY_TOKEN` in workflow files
2. **Use dry-run in PRs** — Validate publishability without spending your one-shot crates.io publish
3. **Pin to a specific version tag** — Use `@v1.2.4` instead of `@main` for stability
4. **Run security scan first** — Use [`rust/security-scan`](../security-scan/README.md) before publishing to catch vulnerabilities
5. **Commit `Cargo.lock`** — Ensures reproducible builds and deterministic publish behavior

## Troubleshooting

### `error: failed to authenticate`

Ensure `CARGO_REGISTRY_TOKEN` is set in your repository secrets under **Settings → Secrets and variables → Actions**. Tokens can be generated at [crates.io/settings/tokens](https://crates.io/settings/tokens).

### `error: crate version already uploaded`

Each version can only be published once. Bump the version in `Cargo.toml` (or the workspace root `[workspace.package]` version) before re-publishing.

### `error[E0635]: unknown feature 'edition2024'`

Your crate uses a Rust edition newer than the stable toolchain installed on the runner. Ensure the runner's Rust toolchain matches the edition declared in `Cargo.toml`.

### `error: no matching package named '...'`

Ensure `workspace: true` is set and `package` matches the `name` field in the crate's `Cargo.toml`, not the directory name. The action must run from the workspace root.

## Compatibility

| Component | Requirement |
|-----------|-------------|
| Rust | stable, beta, or nightly |
| Cargo | Any version supporting `--package` |
| OS | Linux, macOS, Windows |
| Runners | ubuntu-latest, macos-latest, windows-latest, self-hosted |
| Registry | crates.io (default) or custom via `cargo-args: --registry` |

## Related Actions

- [`rust/package-crate`](../package-crate/README.md) — Package a crate into a `.crate` archive before publishing
- [`rust/setup-rust-build`](../setup-rust-build/README.md) — Set up Rust toolchain before publishing
- [`rust/security-scan`](../security-scan/README.md) — Scan Cargo dependencies for vulnerabilities
- [`security/cosign-sign`](../../security/cosign-sign/README.md) — Sign release artifacts with Cosign

## License

MIT License — see [LICENSE](../../LICENSE) for details.

## Contributing

Contributions welcome! Please see [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.
