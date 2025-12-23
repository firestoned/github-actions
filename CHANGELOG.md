# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- **rust/package-crate** - Package Rust crates for publishing to crates.io
  - Workspace support with `--package` flag for workspace crates
  - Handles workspace version inheritance (`version.workspace = true`)
  - Configurable allow-dirty flag
  - Additional cargo arguments support
  - Verification output showing packaged `.crate` files
- **rust/publish-crate** - Publish Rust crates to crates.io
  - Workspace support with `--package` flag for workspace crates
  - Handles workspace version inheritance (`version.workspace = true`)
  - Secure token handling (hidden in logs)
  - Dry-run mode for testing
  - Configurable allow-dirty flag
  - Additional cargo arguments support
- **rust/build-library** - Build Rust libraries with flexible profile and feature control
- **rust/lint** - Run cargo fmt and cargo clippy for code quality checks
- **rust/verify-toolchain** - Reusable action to verify Rust toolchain and components are installed
  - Verify cargo, rustfmt, clippy, or llvm-tools-preview
  - Outputs version information for all verified tools
  - Clear, actionable error messages with setup instructions
  - Used internally by other Rust actions (lint, build-binary, build-library, generate-sbom)
- **rust/lint** features:
  - Configurable fmt and clippy checks (can enable/disable individually)
  - Custom clippy arguments and lint levels
  - Feature support (all-features, specific features, no-default-features)
  - Workspace support (entire workspace or specific packages)
  - Fail-on-warnings option
  - Detailed step summaries and outputs
- Comprehensive test coverage for all Rust actions:
  - **rust/cache-cargo** - Cache creation verification
  - **rust/setup-rust-build** - Multi-target setup (x86_64, ARM64), custom components
  - **rust/verify-toolchain** - Component verification, outputs validation, failure scenarios
  - **rust/build-binary** - Binary compilation and execution
  - **rust/build-library** - Dev/release profiles, feature flags
  - **rust/lint** - Well-formatted, badly-formatted, clippy warnings
  - **rust/generate-sbom** - Single crate, workspace, multiple formats (JSON/XML)
  - **rust/package-crate** - Standalone and workspace crate packaging
  - **rust/publish-crate** - Dry-run publishing (actual publish requires token)
  - **rust/security-scan** - cargo-audit integration
- Comprehensive test coverage for security actions:
  - **security/trivy-scan** - Container scanning, SARIF output
  - **security/cosign-sign** - Installation verification
  - **security/license-check** - SPDX header validation
  - **security/verify-signed-commits** - Commit signature verification
- Example library CI workflow demonstrating feature matrix testing
- Release workflow automation (triggers on release published event)
- RELEASE_PROCESS.md documentation for release workflow
- Workspace support for **rust/generate-sbom** action
  - `workspace` input to generate SBOM for entire workspace
  - `package` input to generate SBOM for specific workspace package
  - Enhanced file discovery for workspace-generated SBOMs
  - Complete workspace workflow examples in documentation

### Changed
- **rust/setup-rust-build** now includes `rustfmt` and `clippy` components by default
  - Ensures all Rust tooling is available for linting and code quality checks
  - Added new `components` input parameter for customization (default: `rustfmt, clippy`)
  - Users can customize components or set to empty string for minimal installation
  - No breaking changes - existing workflows continue to work
- **rust/generate-sbom** now supports Cargo workspaces with `--all` flag (cargo-cyclonedx 0.5.7)
  - Changed from `--workspace` to `--all` for cargo-cyclonedx compatibility
  - Enhanced file discovery to check root directory, crate directories, and target directories
  - Uses `compgen -G` for robust file existence checking
- **rust/publish-crate** now uses modern `cargo login` authentication
  - Uses `CARGO_REGISTRY_TOKEN` environment variable instead of deprecated `--token` flag
  - Separate login step for clearer authentication flow
  - Token never exposed in command-line arguments
- **rust/package-crate** now uses `compgen -G` for file verification
  - More robust .crate file detection
  - Better error handling for missing files
- **rust/security-scan** default cargo-audit version updated to 0.22.0
  - Previously 0.21.0
  - Includes latest vulnerability database and bug fixes
- **rust/lint**, **rust/build-binary**, **rust/build-library**, and **rust/generate-sbom** now use **rust/verify-toolchain** action
  - Provides consistent toolchain verification across all Rust actions
  - Reduces code duplication
  - Clearer error messages with actionable guidance
- Updated all Rust action documentation to use `firestoned` organization name
  - Changed all references from `your-org` to `firestoned`
  - Ensures documentation examples work out of the box
- Updated generate-sbom README with workspace examples and best practices
- Updated all Rust action documentation to prioritize `rust/setup-rust-build` over external setup actions
- All file existence checks now use `compgen -G` pattern throughout actions and tests
  - More reliable than `ls` with glob patterns
  - Consistent error handling

## [1.0.0] - 2025-12-18

### Added

#### Rust Ecosystem Actions
- **rust/cache-cargo** - Cache Cargo registry, index, and build artifacts
- **rust/setup-rust-build** - Set up Rust toolchain with cross-compilation support
- **rust/build-binary** - Build Rust binaries for x86_64 and ARM64
- **rust/security-scan** - Scan Rust dependencies for vulnerabilities using cargo-audit
- **rust/generate-sbom** - Generate Software Bill of Materials in CycloneDX format

#### Security & Compliance Actions
- **security/trivy-scan** - Container vulnerability scanning with Trivy and GitHub Security integration
- **security/cosign-sign** - Sign container images and artifacts with Cosign (keyless)
- **security/verify-signed-commits** - Verify all commits are cryptographically signed
- **security/license-check** - Verify SPDX license headers in source files (parameterized)

#### Docker & Container Actions
- **docker/setup-docker** - Set up Docker Buildx and authenticate to GHCR

#### Versioning & Release Actions
- **versioning/extract-version** - Extract version information for consistent tagging (parameterized)

### Documentation
- Comprehensive README.md for repository with quick start guide
- Individual README.md files for all 11 actions (300-700 lines each)
- Usage examples for basic and advanced scenarios
- Best practices and troubleshooting guides
- Complete input/output parameter documentation
- Migration guides for refactored actions

### Testing
- GitHub Actions workflow for testing all actions
- Validation workflows for action syntax and documentation
- License header verification

### Features
- All actions include MIT license headers
- All actions include branding metadata for GitHub Marketplace
- All actions are fully parameterized for maximum reusability
- Support for multiple languages (Rust, Go, Python, Node.js, Java)
- Enterprise-ready with compliance and security focus

### Refactored from bindy Project
- Extracted from [firestoned/bindy](https://github.com/firestoned/bindy) project
- Refactored `extract-version` to accept `repository` input (was hardcoded)
- Refactored `license-check` to accept `copyright-holder` and `license-id` inputs
- Enhanced `setup-rust-build` with `cross-version` input

### Author
**Erick Bourgeois**

---

## Version Tags

### v1.0.0 (2025-12-18)
- Initial stable release
- All 11 actions production-ready
- Complete documentation
- Comprehensive testing

### Version Scheme
- **Major (v1, v2)** - Breaking changes to inputs/outputs
- **Minor (v1.1, v1.2)** - New features, backward compatible
- **Patch (v1.0.1, v1.0.2)** - Bug fixes

---

## Migration Guide

### From bindy/.github/actions to firestoned/github-actions

If you're migrating from the bindy project's local actions to this shared repository:

#### cache-cargo
```yaml
# Before
uses: ./.github/actions/cache-cargo

# After
uses: firestoned/github-actions/rust/cache-cargo@v1
```

#### extract-version (REQUIRES CHANGES)
```yaml
# Before (hardcoded repository)
uses: ./.github/actions/extract-version
with:
  workflow-type: main

# After (parameterized)
uses: firestoned/github-actions/versioning/extract-version@v1
with:
  repository: firestoned/your-project  # NEW REQUIRED INPUT
  workflow-type: main
```

#### license-check (REQUIRES CHANGES)
```yaml
# Before (hardcoded copyright and license)
uses: ./.github/actions/license-check

# After (parameterized)
uses: firestoned/github-actions/security/license-check@v1
with:
  copyright-holder: 'Your Name, your-org'  # NEW REQUIRED INPUT
  license-id: MIT  # NEW INPUT (default: MIT)
```

#### All Other Actions
All other actions can be migrated with a simple path change:

```yaml
# Pattern
uses: ./.github/actions/<action-name>
# Becomes
uses: firestoned/github-actions/<category>/<action-name>@v1
```

---

## Roadmap

### Future Enhancements

#### Planned for v1.1.0
- [ ] Add `go/` actions category for Go projects
- [ ] Add `python/` actions category for Python projects
- [ ] Add `node/` actions category for Node.js projects
- [ ] Enhanced SBOM generation for multi-language projects
- [ ] Dependabot configuration examples

#### Planned for v1.2.0
- [ ] Add `kubernetes/` actions for K8s deployment
- [ ] Add `terraform/` actions for infrastructure as code
- [ ] Add `helm/` actions for Helm chart operations
- [ ] Enhanced security scanning with multiple tools

#### Under Consideration
- [ ] GitHub Marketplace publication
- [ ] Action usage analytics
- [ ] Auto-update bot for dependents
- [ ] Action performance benchmarks
- [ ] Community contribution templates

---

## Breaking Changes

### None (v1.0.0 is first release)

Future breaking changes will be documented here with migration guides.

---

## Deprecation Policy

### Deprecation Notice Period
- Minimum 3 months notice before deprecation
- Deprecation warnings in action outputs
- Migration guides provided
- Support period of 6 months after deprecation

### No Current Deprecations

---

## Contributors

### Core Team
- **Erick Bourgeois** - Initial work, all actions, documentation

### Contributing
See [CONTRIBUTING.md](CONTRIBUTING.md) for how to contribute.

---

## Acknowledgments

- Extracted from [firestoned/bindy](https://github.com/firestoned/bindy) project
- Built for enterprise environments (banking, finance, healthcare)
- Battle-tested in production Kubernetes clusters
- Designed for regulatory compliance (SOC2, PCI-DSS, HIPAA)

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Date:** 2025-12-18
**Version:** 1.0.0
**Author:** Erick Bourgeois
