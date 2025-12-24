# Firestoned GitHub Actions - Development Guidelines

This document provides comprehensive guidance for developing, maintaining, and contributing to this GitHub Actions repository.

## Critical Requirements

**CRITICAL**: Any time we make any changes to the GitHub Actions, make sure the readme/docs are fully in sync and we have test cases for each one.

## Project Overview

**Repository**: Firestoned GitHub Actions - A collection of production-ready, reusable GitHub Actions composite workflows
**Organization**: Firestoned (enterprise-focused)
**Primary Language**: Bash with YAML configuration
**Focus**: Rust, Security/Compliance, Docker, and Versioning
**License**: MIT
**Status**: Active development with 16 actions across 5 categories

## Repository Structure

```
github-actions/
├── rust/                    # 10 Rust-specific actions
│   ├── cache-cargo
│   ├── setup-rust-build
│   ├── verify-toolchain
│   ├── build-binary
│   ├── build-library
│   ├── lint
│   ├── security-scan
│   ├── generate-sbom
│   ├── package-crate
│   └── publish-crate
├── security/               # 4 Security & Compliance actions
│   ├── license-check
│   ├── verify-signed-commits
│   ├── trivy-scan
│   └── cosign-sign
├── docker/                 # 1 Docker action
│   └── setup-docker
├── versioning/             # 1 Versioning action
│   └── extract-version
├── examples/               # Example workflows
├── .github/workflows/      # CI/CD automation
│   ├── pr.yml             # Pull request testing workflow
│   └── release.yml        # Release automation workflow
```

## Action Design Principles

### Core Principles

1. **Zero Hardcoding**: All values must be input parameters - no hardcoded paths or values
2. **Sensible Defaults**: Optional inputs must have practical, production-ready defaults
3. **Composability**: Actions must chain together naturally for complex workflows
4. **Security First**: No secret logging, strict input validation, least privilege principle
5. **Enterprise Ready**: SBOM generation, compliance tracking, audit trails
6. **Multi-platform**: Support for Linux, macOS, Windows where applicable
7. **Language Agnostic**: Design actions to work with any language when possible
8. **Fail-Fast**: Clear error messages with helpful guidance for resolution
9. **Comprehensive Documentation**: Every action requires complete documentation

### Mandatory Action Components

Every action MUST include:

1. **action.yml/action.yaml** with:
   - SPDX license header in first 10 lines
   - Clear name and description
   - All inputs with descriptions and defaults
   - All outputs with descriptions
   - Proper branding configuration

2. **README.md** (300-700 lines) with:
   - Title and brief description (1-2 paragraphs)
   - Features list (bullet points)
   - Usage section (basic and complete examples)
   - Inputs table (all parameters documented)
   - Outputs table (if applicable)
   - Examples section (minimum 5 scenarios)
   - How It Works section (technical details)
   - Best Practices section
   - Troubleshooting section
   - Advanced Usage section
   - Compatibility section
   - Related Actions section
   - License section
   - Contributing link

3. **Test Coverage** in `.github/workflows/pr.yml`:
   - Functional tests with realistic scenarios
   - Matrix testing for multiple configurations
   - Validation tests for syntax and required fields
   - Negative testing (actions fail when they should)
   - Output verification

## Coding Standards

### YAML (action.yml)

```yaml
# Copyright header with SPDX license identifier (REQUIRED in first 10 lines)
# Copyright (c) 2025 Erick Bourgeois, firestoned
# SPDX-License-Identifier: MIT

name: 'Action Name'
description: 'Clear, concise description'
author: 'Author Name'

branding:
  icon: 'icon-name'      # From Feather icons
  color: 'color'         # blue, green, orange, red, purple, gray

inputs:
  input-name:
    description: 'Input description'
    required: true/false
    default: 'default-value'

outputs:
  output-name:
    description: 'Output description'
    value: ${{ steps.step-id.outputs.value }}

runs:
  using: composite
  steps:
    # Implementation in Bash
```

**YAML Standards**:
- 2-space indentation
- Quoted strings for all values
- kebab-case for inputs, outputs, and step IDs
- No trailing whitespace
- Newline at end of file

### Bash Scripts

**Required Pattern**:
```bash
#!/usr/bin/env bash
set -euo pipefail

# Always quote variables
echo "Value: ${VARIABLE}"

# Use descriptive names
rust_target="${{ inputs.target }}"

# Provide clear error messages
if [ ! -f "Cargo.toml" ]; then
  echo "Error: Cargo.toml not found in current directory"
  echo "Please run this action from a Rust project root"
  exit 1
fi
```

**Bash Standards**:
- Strict mode: `set -euo pipefail`
- Quote all variables: `"${VARIABLE}"`
- Descriptive variable names (no single letters except loop counters)
- Clear error messages with resolution guidance
- Use `compgen` for command/binary availability checks
- Proper exit codes (0 for success, non-zero for failure)

### Markdown Documentation

**Standards**:
- ATX-style headers (`#` not underlines)
- Fenced code blocks with language tags
- Tables for structured data (inputs, outputs)
- Clear section hierarchy (h2 for main sections, h3 for subsections)
- Code examples for all input combinations
- Cross-references to related actions

## Naming Conventions

- **Input names**: kebab-case (e.g., `copyright-holder`, `cargo-audit-version`)
- **Output names**: kebab-case (e.g., `fmt-status`, `tag-name`)
- **Step IDs**: kebab-case (e.g., `cache-cargo`, `verify-all`)
- **Action directories**: kebab-case category/action-name (e.g., `rust/cache-cargo`)
- **Variables**: snake_case in bash scripts (e.g., `rust_target`, `cargo_version`)

## Common Patterns

### Caching Pattern

```yaml
- name: Cache tool binary
  uses: actions/cache@v4
  with:
    path: ~/.cargo/bin/tool-name
    key: ${{ runner.os }}-tool-name-${{ inputs.tool-version }}
```

### Step Summary Pattern

```yaml
- name: Generate step summary
  shell: bash
  run: |
    echo "### Action Results" >> $GITHUB_STEP_SUMMARY
    echo "| Metric | Value |" >> $GITHUB_STEP_SUMMARY
    echo "|--------|-------|" >> $GITHUB_STEP_SUMMARY
    echo "| Status | ✅ Success |" >> $GITHUB_STEP_SUMMARY
```

### Output Pattern

```yaml
outputs:
  output-name:
    description: 'Output description'
    value: ${{ steps.step-id.outputs.value }}

# In the step:
- name: Set output
  id: step-id
  shell: bash
  run: |
    echo "value=result" >> $GITHUB_OUTPUT
```

### Conditional Execution Pattern

```yaml
- name: Optional step
  if: inputs.enable-feature == 'true'
  shell: bash
  run: |
    # Feature implementation
```

## Testing Requirements

### Test Coverage Requirements

Every action MUST have:
1. At least one functional test in `.github/workflows/pr.yml`
2. Tests for all major input combinations
3. Negative tests (failure scenarios)
4. Output verification
5. Matrix testing for multi-configuration actions

### Test Pattern

```yaml
test-action-name:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      include:
        - config: value1
          description: 'Test case 1'
        - config: value2
          description: 'Test case 2'
  steps:
    - uses: actions/checkout@v4

    - name: Test action
      uses: ./path/to/action
      with:
        input: ${{ matrix.config }}

    - name: Verify results
      run: |
        # Verification logic
```

## Commit Message Standards

Follow conventional commits format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `test`: Adding or updating tests
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `chore`: Changes to build process or auxiliary tools

**Examples**:
```
feat(rust): add cargo-deny security scanning
fix(security): correct Trivy SARIF output format
docs(readme): improve installation instructions
test(lint): add test for clippy warnings
```

## Pull Request Requirements

Before submitting a PR, ensure:

1. ✅ All commits are signed (GPG/SSH)
2. ✅ All tests pass locally
3. ✅ Documentation updated (README.md)
4. ✅ CHANGELOG.md updated
5. ✅ SPDX headers present in all action.yml files
6. ✅ Action README.md is comprehensive (300+ lines)
7. ✅ Test cases added to `.github/workflows/pr.yml`
8. ✅ No hardcoded values (all configurable via inputs)
9. ✅ Sensible defaults for optional inputs
10. ✅ Clear error messages with resolution guidance

## Versioning Strategy

### Semantic Versioning

- **Major (v1, v2)**: Breaking changes to inputs/outputs
- **Minor (v1.1, v1.2)**: New features, backward compatible
- **Patch (v1.0.1, v1.0.2)**: Bug fixes

### Version Tags

- `v1` - Latest v1.x.x (auto-updates, recommended for most users)
- `v1.0` - Latest v1.0.x (patch updates only)
- `v1.0.0` - Exact version (no auto-updates, for strict pinning)

### When to Bump Versions

- **Major**: Renaming inputs, removing inputs, changing output format
- **Minor**: Adding new inputs, new outputs, new features
- **Patch**: Bug fixes, documentation updates, performance improvements

## Security Guidelines

1. **Never log secrets**: Use `::add-mask::` for sensitive values
2. **Input validation**: Validate all user inputs before use
3. **Least privilege**: Request minimum permissions needed
4. **Dependency pinning**: Pin all action dependencies to exact versions or SHA
5. **SPDX compliance**: All files must have SPDX headers
6. **Signed commits**: All commits must be GPG or SSH signed
7. **Security scanning**: All actions undergo security analysis

## Supported Build Targets

The Rust actions support building binaries for multiple target architectures:

### Linux Targets

**Native Builds (using `cargo`):**
- `x86_64-unknown-linux-gnu` - x86_64 Linux with GNU libc
- `x86_64-unknown-linux-musl` - x86_64 Linux with musl libc

**Cross-Compilation Builds (using `cross`):**
- `aarch64-unknown-linux-gnu` - ARM64 Linux with GNU libc
- `aarch64-unknown-linux-musl` - ARM64 Linux with musl libc

### Windows Targets

**Cross-Compilation from Linux (using `cargo`):**
- `x86_64-pc-windows-msvc` - x86_64 Windows with MSVC toolchain (recommended)
- `x86_64-pc-windows-gnu` - x86_64 Windows with GNU/MinGW-w64 toolchain

**Cross-Compilation from Linux (using `cross`):**
- `aarch64-pc-windows-msvc` - ARM64 Windows with MSVC toolchain

**Choosing Between Windows MSVC and GNU:**

MSVC (`x86_64-pc-windows-msvc`):
- Uses Microsoft Visual C++ toolchain
- Better integration with Windows ecosystem
- Best compatibility with Windows APIs
- **Requires Windows runners** (`runs-on: windows-latest`)
- Cannot cross-compile from Linux without complex setup (xwin)
- Recommended for production Windows deployments

GNU (`x86_64-pc-windows-gnu`) - **Recommended for CI/CD**:
- Uses MinGW-w64 toolchain
- **Works on Linux runners** with mingw-w64 installed
- Better for cross-platform CI pipelines on Linux runners
- Requires mingw-w64 installation: `sudo apt-get install mingw-w64`
- Good compatibility with most Windows applications
- Simpler and faster for Linux-based CI/CD

**CI/CD Recommendation**:
- Use `x86_64-pc-windows-gnu` on Linux runners (ubuntu-latest)
- Use `x86_64-pc-windows-msvc` on Windows runners (windows-latest)
- Both produce fully functional Windows executables

### macOS Targets

- `x86_64-apple-darwin` - x86_64 macOS (Intel)
- `aarch64-apple-darwin` - ARM64 macOS (Apple Silicon)

### Target-Specific Requirements

**Windows GNU Target (`x86_64-pc-windows-gnu`):**
```yaml
- name: Install mingw-w64
  if: matrix.target == 'x86_64-pc-windows-gnu'
  run: |
    sudo apt-get update
    sudo apt-get install -y mingw-w64
```

**Cross-compilation targets (require Docker):**
- `aarch64-unknown-linux-gnu` - ARM64 Linux
- `aarch64-pc-windows-msvc` - ARM64 Windows

These targets automatically use the `cross` tool which requires Docker to be available on the runner.

### Binary Output Paths

**Linux binaries:**
```
target/{target}/release/{binary-name}
Example: target/x86_64-unknown-linux-gnu/release/my-app
```

**Windows binaries (include .exe extension):**
```
target/{target}/release/{binary-name}.exe
Example: target/x86_64-pc-windows-msvc/release/my-app.exe
```

## Technologies and Tools

### Core Technologies

- **Bash**: All action implementations
- **YAML**: Action definitions and workflows
- **Git**: Version control and commit verification
- **Docker**: Container operations and cross-compilation

### Rust-Specific Tools

- **Cargo**: Package management and building
- **cargo-audit**: Vulnerability scanning
- **cargo-cyclonedx**: SBOM generation
- **cross**: ARM64 cross-compilation
- **rustfmt**: Code formatting
- **clippy**: Linting

### Security Tools

- **Trivy**: Container vulnerability scanning
- **Cosign**: Container image signing
- **SPDX**: License identification standard
- **CycloneDX**: SBOM specification

### GitHub Actions Dependencies

- `actions/checkout@v4` - Code checkout
- `actions/cache@v4` - Artifact caching
- `dtolnay/rust-toolchain` - Rust setup
- `Swatinem/rust-cache@v2` - Rust dependency caching

## Documentation Synchronization Checklist

When making changes to any action, verify:

- [ ] `action.yml` inputs/outputs match README.md documentation
- [ ] All examples in README.md are tested and work
- [ ] CHANGELOG.md updated with changes
- [ ] Test cases in `.github/workflows/pr.yml` cover new functionality
- [ ] Related actions documentation cross-referenced
- [ ] Main README.md updated if action list or features changed
- [ ] Examples directory updated if workflow patterns changed

## Best Practices

### Error Handling

```bash
# Good: Clear error with guidance
if [ ! -f "Cargo.toml" ]; then
  echo "Error: Cargo.toml not found"
  echo "Please run this action from a Rust project root"
  exit 1
fi

# Bad: Unclear error
if [ ! -f "Cargo.toml" ]; then
  echo "File not found"
  exit 1
fi
```

### Input Validation

```bash
# Validate required inputs
if [ -z "${INPUT_VALUE}" ]; then
  echo "Error: input 'value' is required but not provided"
  exit 1
fi

# Validate enum inputs
case "${INPUT_FORMAT}" in
  json|yaml|xml)
    # Valid
    ;;
  *)
    echo "Error: format must be one of: json, yaml, xml"
    exit 1
    ;;
esac
```

### Caching Strategy

```yaml
# Cache expensive operations
- name: Cache cargo-audit
  uses: actions/cache@v4
  with:
    path: ~/.cargo/bin/cargo-audit
    key: ${{ runner.os }}-cargo-audit-${{ inputs.cargo-audit-version }}

# Use cache to avoid reinstalling tools
- name: Install cargo-audit
  if: steps.cache.outputs.cache-hit != 'true'
  shell: bash
  run: cargo install cargo-audit --version "${{ inputs.cargo-audit-version }}"
```

### Tool Installation Pattern

```bash
# Check if tool exists
if ! command -v tool-name >/dev/null 2>&1; then
  echo "Installing tool-name..."
  # Installation logic
else
  echo "tool-name already installed"
fi
```

## Related Documentation

- [CONTRIBUTING.md](CONTRIBUTING.md) - Contribution guidelines
- [README.md](README.md) - Main project documentation
- [CHANGELOG.md](CHANGELOG.md) - Version history
- [examples/README.md](examples/README.md) - Example workflows

## License

This project is licensed under the MIT License. All files must include SPDX license identifiers.

```yaml
# Copyright (c) 2025 Erick Bourgeois, firestoned
# SPDX-License-Identifier: MIT
```
