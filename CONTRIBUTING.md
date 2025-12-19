# Contributing to Firestoned GitHub Actions

Thank you for your interest in contributing! This document provides guidelines for contributing to the firestoned/github-actions repository.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Adding a New Action](#adding-a-new-action)
- [Modifying Existing Actions](#modifying-existing-actions)
- [Documentation Standards](#documentation-standards)
- [Testing](#testing)
- [Pull Request Process](#pull-request-process)
- [Versioning](#versioning)
- [Style Guide](#style-guide)

---

## Code of Conduct

### Our Standards

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Focus on constructive feedback
- Assume good intentions

### Enforcement

Violations of the code of conduct can be reported to the repository maintainers.

---

## How to Contribute

### Types of Contributions

1. **Bug Fixes** - Fix issues in existing actions
2. **New Actions** - Add new composite actions
3. **Documentation** - Improve or add documentation
4. **Examples** - Add usage examples
5. **Tests** - Improve test coverage
6. **Feature Enhancements** - Enhance existing actions

### Before You Start

1. **Check existing issues** - See if your idea is already being discussed
2. **Open an issue first** - For major changes, discuss before implementing
3. **Small PRs are better** - Break large changes into smaller, focused PRs

---

## Development Setup

### Prerequisites

- Git
- GitHub account
- Text editor (VS Code recommended)
- Basic knowledge of GitHub Actions
- For testing: Docker, appropriate language toolchains

### Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR-USERNAME/github-actions.git
cd github-actions

# Add upstream remote
git remote add upstream https://github.com/firestoned/github-actions.git
```

### Keep Your Fork Updated

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

---

## Adding a New Action

### Directory Structure

Create a new action following this structure:

```
<category>/
└── <action-name>/
    ├── action.yml       # Action definition
    └── README.md        # Documentation
```

### Categories

Choose the appropriate category:
- `rust/` - Rust-specific actions
- `security/` - Security and compliance actions
- `docker/` - Docker and container actions
- `versioning/` - Versioning and release actions
- `go/` - Go-specific actions (future)
- `python/` - Python-specific actions (future)
- `node/` - Node.js-specific actions (future)

### Action Template

```yaml
# Copyright (c) 2025 Erick Bourgeois, firestoned
# SPDX-License-Identifier: MIT

name: 'Action Name'
description: 'Brief description of what this action does'
author: 'Your Name'

branding:
  icon: 'icon-name'  # From Feather icons
  color: 'color'     # blue, green, orange, red, purple, gray

inputs:
  input-name:
    description: 'Input description'
    required: true
    default: 'default-value'

outputs:
  output-name:
    description: 'Output description'
    value: ${{ steps.step-id.outputs.value }}

runs:
  using: composite
  steps:
    - name: Step name
      id: step-id
      shell: bash
      run: |
        # Implementation
        echo "output-name=value" >> $GITHUB_OUTPUT
```

### README Template

Every action MUST have a comprehensive README.md with:

1. **Title and Description** (1-2 paragraphs)
2. **Features** (bullet list)
3. **Usage** (basic example)
4. **Inputs** (table with all parameters)
5. **Outputs** (table if applicable)
6. **Examples** (multiple scenarios)
7. **How It Works** (technical details)
8. **Best Practices** (recommendations)
9. **Troubleshooting** (common issues)
10. **Advanced Usage** (power user features)
11. **Compatibility** (OS, runners, versions)
12. **Related Actions** (links to related actions)
13. **License** (MIT)
14. **Contributing** (link to this file)

See existing action READMEs for examples.

---

## Modifying Existing Actions

### Breaking vs. Non-Breaking Changes

**Non-Breaking Changes** (patch or minor version):
- Adding new optional inputs with defaults
- Adding new outputs
- Bug fixes
- Documentation improvements
- Internal implementation changes

**Breaking Changes** (major version):
- Removing inputs or outputs
- Changing input/output behavior significantly
- Removing support for a platform
- Changing required inputs to have no default

### Testing Changes

Before submitting a PR for changes to an existing action:

1. **Test locally** - Create a test repository and workflow
2. **Test in CI** - Use the test workflow in `.github/workflows/test-actions.yml`
3. **Update tests** - Add/update tests for your changes
4. **Update docs** - Reflect changes in README.md
5. **Update changelog** - Add entry to CHANGELOG.md

---

## Documentation Standards

### Action Documentation

Each action's README.md must:
- Be 300+ lines of comprehensive documentation
- Include at least 5 usage examples
- Document all inputs and outputs in tables
- Include troubleshooting section
- Follow markdown best practices
- Use proper code syntax highlighting

### Code Comments

- Comment complex bash scripts
- Explain non-obvious logic
- Reference GitHub Actions documentation when using advanced features

### SPDX Headers

All action.yml files MUST include:

```yaml
# Copyright (c) 2025 Your Name, organization
# SPDX-License-Identifier: MIT
```

---

## Testing

### Local Testing

Test actions locally before submitting:

```bash
# Create a test repository
mkdir test-repo
cd test-repo
git init

# Create a workflow using your action
cat > .github/workflows/test.yml <<EOF
name: Test
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: path/to/your/action
        with:
          input: value
EOF

# Commit and push to trigger
git add .
git commit -m "Test action"
git push
```

### CI Testing

The repository includes a test workflow at `.github/workflows/test-actions.yml`.

Add tests for your action:

```yaml
test-your-action:
  name: Test category/your-action
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Test your action
      uses: ./category/your-action
      with:
        input: test-value

    - name: Verify output
      run: |
        # Verify the action worked correctly
        echo "Verification logic here"
```

### Test Coverage

Aim for:
- All input combinations tested
- All outputs verified
- Error conditions tested
- Multiple OS if applicable (ubuntu, macos, windows)

---

## Pull Request Process

### Before Submitting

- [ ] Code follows style guide
- [ ] All tests pass locally
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] SPDX headers present
- [ ] README.md is comprehensive
- [ ] Commits are signed (GPG/SSH)

### PR Title Format

```
<type>(<scope>): <description>

Examples:
feat(rust): add cargo-deny security scanning
fix(security): correct Trivy SARIF output format
docs(readme): improve installation instructions
test(ci): add tests for extract-version action
```

Types:
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation only
- `test` - Testing improvements
- `refactor` - Code refactoring
- `chore` - Maintenance tasks

### PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New action
- [ ] Enhancement to existing action
- [ ] Documentation update
- [ ] Test improvement

## Testing
Describe testing performed

## Checklist
- [ ] Code follows style guide
- [ ] Documentation updated
- [ ] Tests added/updated
- [ ] CHANGELOG.md updated
- [ ] All tests pass
- [ ] Commits signed

## Related Issues
Closes #123
```

### Review Process

1. **Automated checks** - CI must pass
2. **Maintainer review** - At least one maintainer approval required
3. **Discussion** - Address feedback and questions
4. **Approval** - Maintainer approves and merges

---

## Versioning

### Semantic Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **Major (v1, v2)** - Breaking changes
- **Minor (v1.1, v1.2)** - New features, backward compatible
- **Patch (v1.0.1, v1.0.2)** - Bug fixes

### Version Tags

Maintainers create tags after merging:

```bash
# Tag exact version
git tag v1.2.3

# Update major version tag
git tag -f v1

# Update minor version tag
git tag -f v1.2

# Push tags
git push origin v1.2.3 v1.2 v1 --force
```

### Changelog Updates

Add entries to CHANGELOG.md under `[Unreleased]`:

```markdown
## [Unreleased]

### Added
- New feature description

### Changed
- Changed feature description

### Fixed
- Bug fix description

### Deprecated
- Deprecated feature description
```

Maintainers will move entries to versioned sections on release.

---

## Style Guide

### YAML

```yaml
# Use 2-space indentation
name: 'Action Name'

# Quote strings
description: 'This is a description'

# Use kebab-case for input/output names
inputs:
  my-input-name:
    description: 'Input description'
```

### Bash Scripts

```bash
# Use strict mode
set -euo pipefail

# Quote variables
echo "Value: ${VARIABLE}"

# Use descriptive variable names
USER_NAME="value"  # Good
un="value"         # Bad

# Add comments for complex logic
# Calculate the checksum of the file
CHECKSUM=$(sha256sum file.txt | cut -d' ' -f1)
```

### Markdown

```markdown
# Use ATX-style headers
## Second Level

# Use fenced code blocks with language
```bash
echo "example"
```

# Use tables for structured data
| Column 1 | Column 2 |
|----------|----------|
| Value    | Value    |
```

### Commit Messages

```
<type>(<scope>): <subject>

<body>

<footer>

Example:
feat(rust): add cargo-deny security scanning

This adds a new action for scanning Rust dependencies
using cargo-deny, which checks for security vulnerabilities,
license compliance, and banned dependencies.

Closes #42
```

---

## Action Design Principles

### 1. Zero Hardcoding

All values must be configurable via inputs:

```yaml
# ❌ BAD - Hardcoded value
run: echo "firestoned/myapp"

# ✅ GOOD - Parameterized
run: echo "${{ inputs.repository }}"
```

### 2. Sensible Defaults

Provide defaults for optional inputs:

```yaml
inputs:
  format:
    description: 'Output format'
    required: false
    default: 'json'  # Good default
```

### 3. Clear Error Messages

Fail fast with helpful error messages:

```bash
if [ -z "${{ inputs.required-input }}" ]; then
  echo "ERROR: required-input is mandatory"
  echo "Usage: uses: action@v1"
  echo "       with:"
  echo "         required-input: value"
  exit 1
fi
```

### 4. Comprehensive Documentation

Every action needs:
- Clear description
- Usage examples (basic + advanced)
- Input/output tables
- Troubleshooting guide
- Best practices

### 5. Composability

Actions should work well together:

```yaml
# Should be able to chain actions
- uses: firestoned/github-actions/rust/setup-rust-build@v1
- uses: firestoned/github-actions/rust/build-binary@v1
- uses: firestoned/github-actions/rust/security-scan@v1
```

### 6. Security First

- Never log secrets
- Validate all inputs
- Use principle of least privilege
- Document security implications

---

## Getting Help

### Questions?

- Open a [GitHub Discussion](https://github.com/firestoned/github-actions/discussions)
- Check existing [Issues](https://github.com/firestoned/github-actions/issues)
- Review action documentation in README files

### Found a Bug?

1. Check if it's already reported in [Issues](https://github.com/firestoned/github-actions/issues)
2. Create a new issue with:
   - Action name and version
   - Steps to reproduce
   - Expected vs. actual behavior
   - Workflow YAML (sanitized)

---

## Recognition

Contributors will be:
- Listed in CHANGELOG.md
- Credited in release notes
- Acknowledged in the repository

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to firestoned/github-actions!**
