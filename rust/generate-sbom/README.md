# Generate SBOM

A composite GitHub Action that generates Software Bill of Materials (SBOM) for Rust projects in CycloneDX format, supporting multiple output formats and target architectures.

## Features

- Generates CycloneDX-compliant SBOM files
- Supports JSON and XML output formats
- Three describe modes: entire crate, binaries only, or all Cargo targets
- Multi-architecture support with target specification
- Caches `cargo-cyclonedx` binary for fast execution
- Automatic file discovery and verification
- Industry-standard format for supply chain security

## Usage

### Basic Example (JSON)

```yaml
- name: Generate SBOM
  uses: your-org/github-actions/rust/generate-sbom@v1
```

### XML Format

```yaml
- name: Generate SBOM (XML)
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    format: xml
```

### Both JSON and XML

```yaml
- name: Generate SBOM (both formats)
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    format: both
```

### Describe Entire Crate (Default)

```yaml
- name: Generate SBOM for entire crate
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    describe: crate
```

### Describe Only Binaries

```yaml
- name: Generate SBOM for binaries only
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    describe: binaries
```

### Describe All Cargo Targets

```yaml
- name: Generate SBOM for all targets
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    describe: all-cargo-targets
```

### Target-Specific SBOM

```yaml
- name: Generate SBOM for ARM64
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    target: aarch64-unknown-linux-gnu
    describe: binaries
```

### Workspace - Entire Workspace

```yaml
- name: Generate SBOM for entire workspace
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    workspace: true
    format: both
```

### Workspace - Specific Package

```yaml
- name: Generate SBOM for specific workspace package
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    package: my-core-library
    format: json
```

### Complete Workflow with Upload

```yaml
jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: your-org/github-actions/rust/setup-rust-build@v1
        with:
          target: x86_64-unknown-linux-gnu

      - name: Build
        uses: your-org/github-actions/rust/build-binary@v1
        with:
          target: x86_64-unknown-linux-gnu

      - name: Generate SBOM
        uses: your-org/github-actions/rust/generate-sbom@v1
        with:
          format: both
          describe: crate
          target: x86_64-unknown-linux-gnu

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom-files
          path: |
            *.cdx.json
            *.cdx.xml
            target/**/release/*.cdx.*
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `format` | Output format: `json`, `xml`, or `both` | No | `json` |
| `cyclonedx-version` | Version of `cargo-cyclonedx` to use | No | `0.5.7` |
| `describe` | What to describe: `crate` (entire crate with targets as subcomponents), `binaries` (separate SBOM per binary), or `all-cargo-targets` (separate SBOM per Cargo target) | No | `crate` |
| `target` | Rust target triple (e.g., `x86_64-unknown-linux-gnu`) | No | `''` (default target) |
| `package` | Package to generate SBOM for (for workspaces with multiple packages) | No | `''` |
| `workspace` | Generate SBOM for all workspace members | No | `false` |

### Describe Mode Details

| Mode | Description | Use Case |
|------|-------------|----------|
| `crate` | Entire crate in a single SBOM file, with Cargo targets as subcomponents | Default; most comprehensive single-file SBOM |
| `binaries` | Separate SBOM for each binary (bin, cdylib); ignores other targets | Focus on executable artifacts only |
| `all-cargo-targets` | Separate SBOM for each Cargo target, including non-executable ones (e.g., rlib) | Detailed analysis of all build outputs |

## Output Files

### File Naming Conventions

Generated SBOM files follow these naming patterns:

| Describe | Target | Format | Filename |
|----------|--------|--------|----------|
| `crate` | Default | JSON | `{crate-name}_*.cdx.json` |
| `crate` | Default | XML | `{crate-name}_*.cdx.xml` |
| `binaries` | Default | JSON | `{binary-name}_*.cdx.json` (per binary) |
| `binaries` | Specified | JSON | `target/{target}/release/{binary}_*.cdx.json` |
| `all-cargo-targets` | Default | JSON | `{target-name}_*.cdx.json` (per target) |

### File Location

- **Default target**: Root directory (`./`)
- **Specific target**: `target/{target}/release/`

### Example Output Structure

```
.
├── my-app_1.0.0.cdx.json          # Binary SBOM (JSON)
├── my-app_1.0.0.cdx.xml           # Binary SBOM (XML)
└── target/
    └── x86_64-unknown-linux-gnu/
        └── release/
            ├── my-app_1.0.0.cdx.json
            └── my-app              # The binary
```

## How It Works

### Generation Process

1. **Check cache**: Look for cached `cargo-cyclonedx` binary
2. **Install tool**: Only if not cached
3. **Build command**: Construct command based on inputs
4. **Generate SBOM**: Run `cargo cyclonedx` with specified options
5. **Verify output**: List generated files for confirmation

### Command Construction

The action builds commands based on your configuration:

**Single crate (default)**:
```bash
cargo cyclonedx --all \
  [--target <target>] \
  --describe <crate|binaries|all-cargo-targets> \
  --format <json|xml>
```

**Entire workspace**:
```bash
cargo cyclonedx --workspace \
  [--target <target>] \
  --describe <crate|binaries|all-cargo-targets> \
  --format <json|xml>
```

**Specific package in workspace**:
```bash
cargo cyclonedx --package <package-name> \
  [--target <target>] \
  --describe <crate|binaries|all-cargo-targets> \
  --format <json|xml>
```

### CycloneDX Format

The generated SBOM includes:
- Component metadata (name, version, type)
- Dependencies and dependency relationships
- License information
- Package URLs (PURL)
- Checksums and hashes
- Build metadata

## Best Practices

### 1. Generate After Build

Always generate SBOM after building binaries:

```yaml
- name: Build
  uses: your-org/github-actions/rust/build-binary@v1
  with:
    target: x86_64-unknown-linux-gnu

- name: Generate SBOM
  uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    target: x86_64-unknown-linux-gnu
```

### 2. Include Both Binaries and Dependencies

For comprehensive supply chain visibility:

```yaml
- uses: your-org/github-actions/rust/generate-sbom@v1
  with:
    describe: both
```

### 3. Generate for All Targets

In matrix builds, generate SBOM for each target:

```yaml
strategy:
  matrix:
    target: [x86_64-unknown-linux-gnu, aarch64-unknown-linux-gnu]
steps:
  - uses: your-org/github-actions/rust/generate-sbom@v1
    with:
      target: ${{ matrix.target }}
```

### 4. Archive SBOM Files

Upload SBOM files as release artifacts:

```yaml
- name: Upload SBOM to release
  uses: softprops/action-gh-release@v1
  with:
    files: |
      *.cdx.json
      *.cdx.xml
```

### 5. Sign SBOM Files

Cryptographically sign SBOMs for authenticity:

```yaml
- name: Generate SBOM
  uses: your-org/github-actions/rust/generate-sbom@v1

- name: Sign SBOM
  uses: your-org/github-actions/security/cosign-sign@v1
  with:
    artifact-path: my-app_1.0.0.cdx.json
```

## Troubleshooting

### No SBOM Files Generated

**Problem**: Action completes but no `.cdx.*` files found

**Solution**: Ensure binaries were built before generating SBOM:
```yaml
- name: Build first
  run: cargo build --release --target ${{ matrix.target }}

- name: Then generate SBOM
  uses: your-org/github-actions/rust/generate-sbom@v1
```

### Wrong File Location

**Problem**: SBOM files not in expected directory

**Solution**: Check the target specification:
- No target: Files in `./`
- With target: Files in `target/{target}/release/`

### cargo-cyclonedx Installation Fails

**Problem**: Tool installation times out or fails

**Solution**:
1. Check version exists: https://crates.io/crates/cargo-cyclonedx
2. Try a different version:
   ```yaml
   with:
     cyclonedx-version: '0.5.6'
   ```

### Incomplete Dependency Information

**Problem**: SBOM missing some dependencies

**Solution**: Use `describe: both` and ensure `Cargo.lock` is committed:
```bash
git add Cargo.lock
git commit -m "Lock dependencies for SBOM"
```

## Advanced Usage

### Custom SBOM Metadata

To add custom metadata, post-process the JSON:

```yaml
- name: Generate SBOM
  uses: your-org/github-actions/rust/generate-sbom@v1

- name: Add metadata
  run: |
    jq '.metadata.authors = ["Your Name <you@example.com>"]' \
      my-app_*.cdx.json > sbom-with-metadata.json
```

### Validate SBOM

Validate the generated SBOM against CycloneDX schema:

```yaml
- name: Validate SBOM
  run: |
    npm install -g @cyclonedx/cyclonedx-cli
    cyclonedx-cli validate --input-file my-app_*.cdx.json
```

### Convert Formats

Convert between JSON and XML:

```yaml
- name: Convert JSON to XML
  run: |
    cargo cyclonedx convert --input my-app.cdx.json --output my-app.cdx.xml
```

### Merge Multiple SBOMs

For multi-binary projects:

```yaml
- name: Merge SBOMs
  run: |
    cargo cyclonedx merge \
      --input binary1.cdx.json \
      --input binary2.cdx.json \
      --output merged.cdx.json
```

### Workspace SBOM Workflow

Complete workflow for Cargo workspaces with multiple packages:

```yaml
name: Generate Workspace SBOMs

on:
  push:
    branches: [main]
  release:
    types: [published]

jobs:
  sbom-workspace:
    name: Generate SBOM for Entire Workspace
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache dependencies
        uses: firestoned/github-actions/rust/cache-cargo@v1

      - name: Build entire workspace
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          workspace: true
          all-features: true

      - name: Generate workspace SBOM
        uses: firestoned/github-actions/rust/generate-sbom@v1
        with:
          workspace: true
          format: both
          describe: crate

      - name: Upload workspace SBOM
        uses: actions/upload-artifact@v4
        with:
          name: workspace-sbom
          path: |
            **/*.cdx.json
            **/*.cdx.xml

  sbom-packages:
    name: Generate SBOM per Package
    runs-on: ubuntu-latest
    strategy:
      matrix:
        package:
          - my-core
          - my-cli
          - my-utils
    steps:
      - uses: actions/checkout@v4

      - name: Cache dependencies
        uses: firestoned/github-actions/rust/cache-cargo@v1

      - name: Build package
        uses: firestoned/github-actions/rust/build-library@v1
        with:
          package: ${{ matrix.package }}
          all-features: true

      - name: Generate package SBOM
        uses: firestoned/github-actions/rust/generate-sbom@v1
        with:
          package: ${{ matrix.package }}
          format: json
          describe: crate

      - name: Upload package SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom-${{ matrix.package }}
          path: "*.cdx.json"
```

## SBOM Use Cases

### 1. Vulnerability Management

Feed SBOM to vulnerability scanners:

```yaml
- name: Generate SBOM
  uses: your-org/github-actions/rust/generate-sbom@v1

- name: Scan SBOM
  uses: anchore/sbom-action/scan@v0
  with:
    sbom: my-app_*.cdx.json
```

### 2. License Compliance

Extract license information:

```bash
jq '.components[].licenses' my-app.cdx.json
```

### 3. Dependency Analysis

Analyze dependency relationships:

```bash
jq '.dependencies' my-app.cdx.json
```

### 4. Supply Chain Security

Track component provenance and integrity:

```bash
jq '.components[] | {name, version, purl, hashes}' my-app.cdx.json
```

## CycloneDX Specification

This action generates SBOMs compliant with:
- **CycloneDX Specification**: v1.4 and v1.5
- **Format**: Standard JSON/XML
- **Schema**: https://cyclonedx.org/schema/

### SBOM Components

Generated SBOM includes:

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.4",
  "version": 1,
  "metadata": {
    "component": {
      "type": "application",
      "name": "my-app",
      "version": "1.0.0"
    }
  },
  "components": [
    {
      "type": "library",
      "name": "dependency-name",
      "version": "1.2.3",
      "purl": "pkg:cargo/dependency-name@1.2.3",
      "licenses": [...],
      "hashes": [...]
    }
  ],
  "dependencies": [...]
}
```

## Performance Tips

### 1. Cache cargo-cyclonedx

Already enabled by default. Saves ~30-60s per run.

### 2. Generate Once Per Target

In matrix builds, avoid regenerating for the same target:

```yaml
strategy:
  matrix:
    target: [x86_64-unknown-linux-gnu, aarch64-unknown-linux-gnu]
steps:
  - name: Generate SBOM (once per target)
    uses: your-org/github-actions/rust/generate-sbom@v1
    with:
      target: ${{ matrix.target }}
```

### 3. Choose Minimal Describe Option

If you only need binary info:

```yaml
with:
  describe: binaries  # Faster than 'both'
```

## Related Actions

- [rust/setup-rust-build](../setup-rust-build/README.md) - Setup Rust environment
- [rust/build-binary](../build-binary/README.md) - Build Rust binaries (required before SBOM)
- [rust/security-scan](../security-scan/README.md) - Scan for vulnerabilities
- [security/cosign-sign](../../security/cosign-sign/README.md) - Sign SBOM files
- [security/trivy-scan](../../security/trivy-scan/README.md) - Scan SBOM for vulnerabilities

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Requires Rust toolchain
- **CycloneDX versions**: 1.4 and 1.5
- **cargo-cyclonedx**: Tested with v0.5.6 and later

## Regulatory Compliance

SBOM generation helps meet requirements from:
- **NTIA Minimum Elements**: Supplier, Component, Relationships
- **Executive Order 14028**: Federal software supply chain security
- **NIST 800-161**: Supply chain risk management
- **EU Cyber Resilience Act**: Software composition transparency

## Examples

### Release Workflow with SBOM

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  release:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        target:
          - x86_64-unknown-linux-gnu
          - aarch64-unknown-linux-gnu
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: your-org/github-actions/rust/setup-rust-build@v1
        with:
          target: ${{ matrix.target }}

      - name: Build
        uses: your-org/github-actions/rust/build-binary@v1
        with:
          target: ${{ matrix.target }}

      - name: Generate SBOM
        uses: your-org/github-actions/rust/generate-sbom@v1
        with:
          format: both
          describe: both
          target: ${{ matrix.target }}

      - name: Sign SBOM
        uses: your-org/github-actions/security/cosign-sign@v1
        with:
          artifact-path: target/${{ matrix.target }}/release/my-app_*.cdx.json

      - name: Upload to release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            target/${{ matrix.target }}/release/my-app
            target/${{ matrix.target }}/release/*.cdx.*
            target/${{ matrix.target }}/release/*.bundle
```

### SBOM Validation Workflow

```yaml
jobs:
  validate-sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate SBOM
        uses: your-org/github-actions/rust/generate-sbom@v1

      - name: Install validator
        run: npm install -g @cyclonedx/cyclonedx-cli

      - name: Validate SBOM
        run: |
          for file in *.cdx.json; do
            echo "Validating $file"
            cyclonedx-cli validate --input-file "$file"
          done
```
