# README Samples - Quality Demonstration

This document shows sample sections from both comprehensive READMEs to demonstrate the quality and depth of documentation.

---

## Extract Version README - Sample Sections

### Features Section
```markdown
## Features

- **Three workflow types** - Specialized handling for main, PR, and release workflows
- **Consistent versioning** - Single source of truth for version information across all jobs
- **Image variant support** - Built-in support for distroless and other image variants
- **Semantic versioning** - Proper semver for releases, dated versions for main branch
- **PR-specific tags** - Clean `pr-NUMBER` format for pull request builds
- **Short SHA extraction** - 7-character commit SHA for tracking
- **Repository-agnostic** - Works with any repository, not hardcoded
```

### How It Works - Workflow Type Main
```markdown
### Workflow Type: `main`

For commits to the main branch:

1. **Version format**: `0.0.0-main.YYYY.MM.DD.RUN_NUMBER`
   - Example: `0.0.0-main.2025.12.17.42`
2. **Tag format**: `main-YYYY.MM.DD`
   - Example: `main-2025.12.17`
3. **Image tag**: Same as tag name
   - Example: `main-2025.12.17`

This ensures:
- Daily builds are versioned consistently
- Same date = same tag (builds on same day overwrite)
- Run number included in version for uniqueness
- Clear indication this is a development build
```

### Migration Section
```markdown
## Migration from Hardcoded Version

### Before (Hardcoded)

```yaml
# Old hardcoded approach - NOT recommended
- name: Set version
  id: version
  run: |
    echo "image=ghcr.io/firestoned/bindy:main-$(date +%Y.%m.%d)" >> $GITHUB_OUTPUT
```

**Problems:**
- Hardcoded repository name
- Inconsistent across workflows
- Manual date formatting
- No variant support
- No proper versioning for releases

### After (Parameterized)

```yaml
# New parameterized approach - RECOMMENDED
- name: Extract version
  id: version
  uses: firestoned/github-actions/versioning/extract-version@v1
  with:
    repository: ${{ github.repository }}
    workflow-type: main
```

**Benefits:**
- Repository name from context
- Consistent across all workflows
- Standardized date format
- Support for variants
- Proper handling of releases and PRs
```

---

## License Check README - Sample Sections

### Features Section
```markdown
## Features

- **SPDX standard compliance** - Validates SPDX-License-Identifier format
- **Multi-language support** - Rust, Shell, Makefiles, GitHub Actions YAML
- **Configurable copyright holder** - Not hardcoded to specific organization
- **Flexible license types** - Support for MIT, Apache-2.0, GPL-3.0, and more
- **Selective file type checking** - Enable/disable checks for specific file types
- **Custom exclusion paths** - Exclude build artifacts, dependencies, etc.
- **Detailed reporting** - Shows missing headers and provides fix instructions
- **CI/CD ready** - Fails build if headers are missing
```

### File Type Documentation
```markdown
#### Rust Files (`.rs`)

**Expected header format:**
```rust
// Copyright (c) 2025 Your Name, your-org
// SPDX-License-Identifier: MIT
```

**Checked when:** `check-rust: 'true'`

**Example:**
```rust
// Copyright (c) 2025 Erick Bourgeois, firestoned
// SPDX-License-Identifier: MIT

//! Module documentation

pub fn hello() {
    println!("Hello, world!");
}
```
```

### SPDX Education Section
```markdown
## Understanding SPDX

### What is SPDX?

**SPDX (Software Package Data Exchange)** is an open standard for communicating software license information.

Benefits:
- **Machine-readable** - Tools can parse and validate
- **Standardized** - Consistent across projects
- **Clear** - Unambiguous license identification
- **Compliant** - Meets open-source requirements

### SPDX Format

```
SPDX-License-Identifier: <IDENTIFIER>
```

Examples:
- `SPDX-License-Identifier: MIT`
- `SPDX-License-Identifier: Apache-2.0`
- `SPDX-License-Identifier: GPL-3.0-or-later`
- `SPDX-License-Identifier: MIT OR Apache-2.0`
```

### Migration Section
```markdown
## Migration from Hardcoded Version

### Before (Hardcoded)

The original action hardcoded the copyright holder and license:

```yaml
# Old action.yaml - NOT flexible
- name: Check for SPDX license identifiers
  shell: bash
  run: |
    echo "Checking for SPDX license headers..."
    # Hardcoded: "Erick Bourgeois, firestoned" and "MIT"
    echo "  // Copyright (c) 2025 Erick Bourgeois, firestoned"
    echo "  // SPDX-License-Identifier: MIT"
```

**Problems:**
- Only works for one specific copyright holder
- Only supports MIT license
- Cannot be reused by other projects
- No configurability

### After (Parameterized)

The refactored action accepts inputs:

```yaml
# New action.yml - Flexible and reusable
- name: Check license headers
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Erick Bourgeois, firestoned'
    license-id: MIT
```

**Benefits:**
- Works for any copyright holder
- Supports all SPDX license identifiers
- Reusable across different projects
- Fully configurable
```

### Best Practices Section
```markdown
## Best Practices

### 1. Use Repository Variables

Store copyright holder and license in repository variables:

```yaml
# Repository Settings → Secrets and Variables → Variables
# COPYRIGHT_HOLDER = "Your Name, your-org"
# LICENSE_ID = "MIT"

- uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: ${{ vars.COPYRIGHT_HOLDER }}
    license-id: ${{ vars.LICENSE_ID }}
```

### 2. Run on Every Commit

Add to your main CI workflow:

```yaml
name: CI

on: [push, pull_request]

jobs:
  license-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: firestoned/github-actions/security/license-check@v1
        with:
          copyright-holder: ${{ vars.COPYRIGHT_HOLDER }}
          license-id: ${{ vars.LICENSE_ID }}
```
```

---

## Documentation Statistics

### Extract Version README
- **Total Lines**: 662
- **Main Sections**: 15
- **Code Examples**: 30+
- **Tables**: 4
- **Complete Workflows**: 5

### License Check README
- **Total Lines**: 724
- **Main Sections**: 18
- **Code Examples**: 40+
- **Tables**: 3
- **Complete Workflows**: 6

---

## Quality Highlights

### Both READMEs Include:

1. **Comprehensive Feature Lists**
   - Clear, bulleted feature descriptions
   - Focus on benefits to users

2. **Progressive Examples**
   - Start with basic usage
   - Progress to advanced scenarios
   - Complete workflow examples

3. **Input/Output Tables**
   - Name, Description, Required, Default
   - Clear, structured information

4. **How It Works Sections**
   - Internal behavior explained
   - Different modes documented
   - Expected outputs shown

5. **Migration Guides**
   - Before/after comparisons
   - Problem statements
   - Benefits clearly listed

6. **Best Practices**
   - Numbered, actionable recommendations
   - Code examples for each practice
   - Clear rationale

7. **Troubleshooting**
   - Common problems identified
   - Clear solutions provided
   - Code examples for fixes

8. **Advanced Usage**
   - Matrix builds
   - Reusable workflows
   - Custom configurations

9. **Related Actions**
   - Cross-linking to other actions
   - Builds a cohesive ecosystem

10. **Compatibility Information**
    - Supported platforms
    - Version requirements
    - Self-hosted runner support

---

## Consistency with Tier 1 Actions

Both READMEs follow the same comprehensive style as the existing Tier 1 actions:

- Similar structure and organization
- Consistent formatting and markdown style
- Same level of detail and depth
- Progressive complexity in examples
- Clear separation of concerns
- Professional tone and presentation

---

## Summary

These comprehensive READMEs transform the Tier 2 actions from internal-only tools to production-ready, reusable GitHub Actions that can be used by any project. The documentation quality matches or exceeds the Tier 1 actions, making them discoverable, understandable, and easy to adopt.
