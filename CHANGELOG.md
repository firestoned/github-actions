# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release of firestoned/github-actions composite actions repository
- 11 production-ready GitHub Actions composite workflows
- Comprehensive documentation for all actions

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
