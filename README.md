# Firestoned GitHub Actions

**Reusable composite GitHub Actions for CI/CD pipelines**

## Project Status

[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Release](https://img.shields.io/github/v/release/firestoned/github-actions)](https://github.com/firestoned/github-actions/releases)
[![GitHub commits since latest release](https://img.shields.io/github/commits-since/firestoned/github-actions/latest)](https://github.com/firestoned/github-actions/commits/main)
[![Last Commit](https://img.shields.io/github/last-commit/firestoned/github-actions)](https://github.com/firestoned/github-actions/commits/main)

## CI/CD Status

[![PR Tests](https://github.com/firestoned/github-actions/workflows/Test%20GitHub%20Actions/badge.svg)](https://github.com/firestoned/github-actions/actions/workflows/pr.yml)
[![Release Workflow](https://github.com/firestoned/github-actions/workflows/Release%20GitHub%20Actions/badge.svg)](https://github.com/firestoned/github-actions/actions/workflows/release.yml)

## Technology & Compatibility

[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Rust](https://img.shields.io/badge/Rust-000000?logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)](https://www.linux.org/)
[![macOS](https://img.shields.io/badge/macOS-000000?logo=apple&logoColor=white)](https://www.apple.com/macos/)

## Security & Compliance

[![SPDX](https://img.shields.io/badge/SPDX-License--Identifier-blue)](https://spdx.dev/)
[![CycloneDX](https://img.shields.io/badge/CycloneDX-SBOM-orange)](https://cyclonedx.org/)
[![Trivy](https://img.shields.io/badge/Trivy-Security%20Scanning-blue)](https://trivy.dev/)
[![Cosign](https://img.shields.io/badge/Cosign-Artifact%20Signing-purple)](https://docs.sigstore.dev/cosign/overview/)

## Community & Support

[![Issues](https://img.shields.io/github/issues/firestoned/github-actions)](https://github.com/firestoned/github-actions/issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/firestoned/github-actions)](https://github.com/firestoned/github-actions/pulls)
[![Contributors](https://img.shields.io/github/contributors/firestoned/github-actions)](https://github.com/firestoned/github-actions/graphs/contributors)
[![Stars](https://img.shields.io/github/stars/firestoned/github-actions?style=social)](https://github.com/firestoned/github-actions/stargazers)

---

A collection of production-ready, reusable GitHub Actions composite workflows for Rust, Docker, security scanning, and compliance. Built for enterprise environments with a focus on security, supply chain integrity, and regulatory compliance.

---

## 📋 Table of Contents

- [Quick Start](#quick-start)
- [Available Actions](#available-actions)
  - [Rust Ecosystem](#rust-ecosystem)
  - [Security & Compliance](#security--compliance)
  - [Docker & Containers](#docker--containers)
  - [Versioning & Release](#versioning--release)
- [Usage Examples](#usage-examples)
- [Versioning Strategy](#versioning-strategy)
- [Contributing](#contributing)
- [License](#license)

---

## 🚀 Quick Start

Use these actions in your workflows by referencing them with the `uses` keyword:

```yaml
# Example: Use Rust caching in your workflow
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache Rust dependencies
        uses: firestoned/github-actions/rust/cache-cargo@v1

      - name: Build
        run: cargo build --release
```

**Pin to major version (`@v1`)** for automatic minor/patch updates, or **pin to exact version (`@v1.0.0`)** for stability.

---

## 📦 Available Actions

### Rust Ecosystem

| Action | Description | Documentation |
|--------|-------------|---------------|
| [`rust/cache-cargo`](rust/cache-cargo/README.md) | Cache Cargo registry, index, and build artifacts | [📖 Docs](rust/cache-cargo/README.md) |
| [`rust/setup-rust-build`](rust/setup-rust-build/README.md) | Set up Rust toolchain with cross-compilation support | [📖 Docs](rust/setup-rust-build/README.md) |
| [`rust/build-binary`](rust/build-binary/README.md) | Build Rust binaries for x86_64 and ARM64 | [📖 Docs](rust/build-binary/README.md) |
| [`rust/build-library`](rust/build-library/README.md) | Build Rust libraries with flexible profile and feature control | [📖 Docs](rust/build-library/README.md) |
| [`rust/lint`](rust/lint/README.md) | Run cargo fmt and cargo clippy for code quality | [📖 Docs](rust/lint/README.md) |
| [`rust/security-scan`](rust/security-scan/README.md) | Scan Rust dependencies for vulnerabilities (cargo-audit) | [📖 Docs](rust/security-scan/README.md) |
| [`rust/generate-sbom`](rust/generate-sbom/README.md) | Generate Software Bill of Materials (CycloneDX) | [📖 Docs](rust/generate-sbom/README.md) |

### Security & Compliance

| Action | Description | Documentation |
|--------|-------------|---------------|
| [`security/trivy-scan`](security/trivy-scan/README.md) | Container vulnerability scanning with Trivy | [📖 Docs](security/trivy-scan/README.md) |
| [`security/cosign-sign`](security/cosign-sign/README.md) | Sign container images and artifacts with Cosign | [📖 Docs](security/cosign-sign/README.md) |
| [`security/verify-signed-commits`](security/verify-signed-commits/README.md) | Verify all commits are cryptographically signed | [📖 Docs](security/verify-signed-commits/README.md) |
| [`security/license-check`](security/license-check/README.md) | Verify SPDX license headers in source files | [📖 Docs](security/license-check/README.md) |

### Docker & Containers

| Action | Description | Documentation |
|--------|-------------|---------------|
| [`docker/setup-docker`](docker/setup-docker/README.md) | Set up Docker Buildx and authenticate to GHCR | [📖 Docs](docker/setup-docker/README.md) |

### Versioning & Release

| Action | Description | Documentation |
|--------|-------------|---------------|
| [`versioning/extract-version`](versioning/extract-version/README.md) | Extract version info for consistent tagging across workflows | [📖 Docs](versioning/extract-version/README.md) |

---

## 💡 Usage Examples

### Complete Rust CI/CD Pipeline

```yaml
name: Rust CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Cache Cargo dependencies
      - uses: firestoned/github-actions/rust/cache-cargo@v1

      # Set up Rust toolchain for x86_64
      - uses: firestoned/github-actions/rust/setup-rust-build@v1
        with:
          target: x86_64-unknown-linux-gnu

      # Build binary
      - uses: firestoned/github-actions/rust/build-binary@v1
        with:
          target: x86_64-unknown-linux-gnu

      # Security scan
      - uses: firestoned/github-actions/rust/security-scan@v1
        with:
          cargo-audit-version: '0.21.0'

      # Generate SBOM
      - uses: firestoned/github-actions/rust/generate-sbom@v1
        with:
          format: json
          describe: binaries
```

### Container Security Workflow

```yaml
name: Container Security

on:
  push:
    branches: [main]

jobs:
  scan-and-sign:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4

      # Build your container image
      - name: Build image
        run: docker build -t myapp:latest .

      # Scan with Trivy
      - uses: firestoned/github-actions/security/trivy-scan@v1
        with:
          image-ref: myapp:latest
          sarif-category: trivy-container-scan

      # Sign with Cosign (keyless)
      - uses: firestoned/github-actions/security/cosign-sign@v1
        with:
          image-digest: ${{ steps.build.outputs.digest }}
          registry: ghcr.io
          repository: myorg/myapp
```

### Compliance & Policy Enforcement

```yaml
name: Compliance Checks

on:
  pull_request:
    branches: [main]

jobs:
  compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Fetch all history for commit verification

      # Verify all commits are signed
      - uses: firestoned/github-actions/security/verify-signed-commits@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          verify-mode: pr
          base-ref: origin/main

      # Check SPDX license headers
      - uses: firestoned/github-actions/security/license-check@v1
        with:
          copyright-holder: "Your Organization"
          license-id: MIT
```

---

## 🔖 Versioning Strategy

This repository follows **semantic versioning** for GitHub Actions:

- **Major version (`v1`, `v2`)**: Breaking changes to inputs/outputs
- **Minor version (`v1.1`, `v1.2`)**: New features, backward compatible
- **Patch version (`v1.0.1`, `v1.0.2`)**: Bug fixes

### Recommended Usage

```yaml
# ✅ Recommended: Pin to major version (get minor/patch updates automatically)
uses: firestoned/github-actions/rust/cache-cargo@v1

# ✅ Conservative: Pin to exact version (manual updates only)
uses: firestoned/github-actions/rust/cache-cargo@v1.0.0

# ⚠️ Not recommended: Use main branch (unstable)
uses: firestoned/github-actions/rust/cache-cargo@main
```

### Version Tags

Each action is tagged with:
- `v1` - Latest v1.x.x release (auto-updates)
- `v1.0` - Latest v1.0.x release (patch updates only)
- `v1.0.0` - Exact release (no auto-updates)

---

## 🎯 Design Principles

These actions are built with the following principles:

1. **Zero Hardcoding**: All values are configurable via inputs
2. **Language Agnostic**: Generic actions work with any language/ecosystem
3. **Security First**: Built for regulated environments (banking, finance, healthcare)
4. **Compliance Ready**: SBOM generation, signed commits, license verification
5. **Well Documented**: Every action has comprehensive README with examples
6. **Tested**: Validated in production across multiple projects
7. **Composable**: Actions work together and with standard GitHub Actions

---

## 🏢 Use Cases

### Perfect For:

- ✅ Rust projects with multi-architecture builds
- ✅ Projects requiring SLSA supply chain security
- ✅ Organizations with compliance requirements (SOC2, PCI-DSS, HIPAA)
- ✅ Teams standardizing CI/CD across multiple repositories
- ✅ Open-source projects with security best practices

### Enterprise Features:

- **Supply Chain Security**: SBOM generation, artifact signing, commit verification
- **Vulnerability Management**: Automated scanning with Trivy and cargo-audit
- **Audit Trail**: All actions log detailed output for compliance reviews
- **Reproducible Builds**: Deterministic caching and versioning

---

## 🛠️ Action Categories

### By Language Support

| Language | Actions Available | Count |
|----------|-------------------|-------|
| **Rust** | cache-cargo, setup-rust-build, build-binary, build-library, lint, security-scan, generate-sbom | 7 |
| **Go** | trivy-scan, cosign-sign, verify-signed-commits, license-check, setup-docker, extract-version | 6 |
| **Python** | trivy-scan, cosign-sign, verify-signed-commits, license-check, setup-docker, extract-version | 6 |
| **Node.js** | trivy-scan, cosign-sign, verify-signed-commits, license-check, setup-docker, extract-version | 6 |
| **Java** | trivy-scan, cosign-sign, verify-signed-commits, license-check, setup-docker, extract-version | 6 |
| **Any** | trivy-scan, cosign-sign, verify-signed-commits, license-check, setup-docker, extract-version | 6 |

### By Use Case

| Use Case | Actions | Benefit |
|----------|---------|---------|
| **Security Scanning** | trivy-scan, security-scan | Detect vulnerabilities early |
| **Supply Chain** | generate-sbom, cosign-sign | SLSA compliance, provenance |
| **Compliance** | license-check, verify-signed-commits | Regulatory requirements |
| **Performance** | cache-cargo, setup-rust-build | Faster builds, lower costs |
| **Multi-Arch** | setup-rust-build, build-binary | ARM64 + x86_64 support |

---

## 📚 Documentation

Each action has its own detailed README with:
- Purpose and use cases
- Input parameters and defaults
- Output values
- Complete usage examples
- Troubleshooting tips
- Best practices

**Browse actions:**
- [Rust Actions](rust/)
- [Security Actions](security/)
- [Docker Actions](docker/)
- [Versioning Actions](versioning/)

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-action`)
3. Follow existing action patterns and documentation style
4. Test your changes thoroughly
5. Submit a pull request

### Adding a New Action

1. Create directory: `<category>/<action-name>/`
2. Add `action.yml` with proper inputs/outputs
3. Create comprehensive `README.md`
4. Update main README.md with new action
5. Add test workflow in `.github/workflows/`

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## 🐛 Reporting Issues

Found a bug or have a feature request?

1. Check [existing issues](https://github.com/firestoned/github-actions/issues)
2. Create a new issue with:
   - Action name and version
   - Steps to reproduce
   - Expected vs. actual behavior
   - Workflow YAML (sanitized)

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

These actions are battle-tested in production environments across:
- Banking and financial services
- Platform engineering teams
- Multi-cluster Kubernetes deployments
- Regulated compliance environments

Built with ❤️ by the firestoned team.

---

## 📞 Support

- **Documentation**: [GitHub Actions Documentation](https://docs.github.com/en/actions)
- **Issues**: [GitHub Issues](https://github.com/firestoned/github-actions/issues)
- **Discussions**: [GitHub Discussions](https://github.com/firestoned/github-actions/discussions)

---

**Star this repo** ⭐ if you find it useful!
