# License Header Check

**Verify SPDX license headers in all source files**

A composite GitHub Action that validates SPDX license identifiers are present in source files across your repository. Ensures compliance with open-source licensing requirements and maintains consistent copyright headers throughout your codebase.

## Features

- **SPDX standard compliance** - Validates SPDX-License-Identifier format
- **Multi-language support** - Rust, Shell, Makefiles, GitHub Actions YAML
- **Configurable copyright holder** - Not hardcoded to specific organization
- **Flexible license types** - Support for MIT, Apache-2.0, GPL-3.0, and more
- **Selective file type checking** - Enable/disable checks for specific file types
- **Custom exclusion paths** - Exclude build artifacts, dependencies, etc.
- **Detailed reporting** - Shows missing headers and provides fix instructions
- **CI/CD ready** - Fails build if headers are missing

## Usage

### Basic Usage

```yaml
- uses: actions/checkout@v4

- name: Check license headers
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Your Name, your-org'
    license-id: MIT
```

### With Apache License

```yaml
- name: Check Apache license headers
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'ACME Corporation'
    license-id: Apache-2.0
```

### With GPL License

```yaml
- name: Check GPL license headers
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Free Software Foundation'
    license-id: GPL-3.0
```

### Selective File Type Checking

```yaml
- name: Check only Rust and Shell files
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Your Name'
    license-id: MIT
    check-rust: 'true'
    check-shell: 'true'
    check-makefiles: 'false'
    check-yaml: 'false'
```

### Custom Exclusion Paths

```yaml
- name: Check with custom exclusions
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Your Name'
    license-id: MIT
    exclude-paths: 'target/,build/,vendor/,node_modules/,.git/'
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `copyright-holder` | Copyright holder name (e.g., "Erick Bourgeois, firestoned") | Yes | N/A |
| `license-id` | SPDX license identifier (e.g., "MIT", "Apache-2.0", "GPL-3.0") | Yes | N/A |
| `check-rust` | Check Rust files (.rs) | No | `true` |
| `check-shell` | Check Shell scripts (.sh, .bash) | No | `true` |
| `check-makefiles` | Check Makefiles (Makefile, *.mk) | No | `true` |
| `check-yaml` | Check GitHub Actions workflows (.yaml, .yml in .github/) | No | `true` |
| `exclude-paths` | Comma-separated paths to exclude | No | `target/,.git/,docs/target/` |

## Outputs

This action does not produce outputs. It either:
- **Succeeds** (exit 0) - All files have proper license headers
- **Fails** (exit 1) - Some files are missing license headers

## How It Works

### Scan Process

1. **Configuration**: Reads inputs for copyright holder, license, and file types
2. **File Discovery**: Finds all source files matching enabled file types
3. **Exclusion Filtering**: Excludes paths specified in `exclude-paths`
4. **Header Validation**: Checks first 10 lines of each file for SPDX identifier
5. **Reporting**: Generates detailed report of missing headers
6. **Exit**: Exits with success or failure based on results

### Supported File Types

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

#### Shell Scripts (`.sh`, `.bash`)

**Expected header format:**
```bash
#!/usr/bin/env bash
# Copyright (c) 2025 Your Name, your-org
# SPDX-License-Identifier: MIT
```

**Checked when:** `check-shell: 'true'`

**Example:**
```bash
#!/usr/bin/env bash
# Copyright (c) 2025 Erick Bourgeois, firestoned
# SPDX-License-Identifier: MIT

set -euo pipefail

echo "Running tests..."
```

#### Makefiles (`Makefile`, `*.mk`)

**Expected header format:**
```makefile
# Copyright (c) 2025 Your Name, your-org
# SPDX-License-Identifier: MIT
```

**Checked when:** `check-makefiles: 'true'`

**Example:**
```makefile
# Copyright (c) 2025 Erick Bourgeois, firestoned
# SPDX-License-Identifier: MIT

.PHONY: build
build:
	cargo build --release
```

#### GitHub Actions (`.yaml`, `.yml`)

**Expected header format:**
```yaml
# Copyright (c) 2025 Your Name, your-org
# SPDX-License-Identifier: MIT
```

**Checked when:** `check-yaml: 'true'`

**Location:** Only checks files in `.github/` directory

**Example:**
```yaml
# Copyright (c) 2025 Erick Bourgeois, firestoned
# SPDX-License-Identifier: MIT

name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

### SPDX License Identifiers

Common SPDX identifiers supported:

| License | SPDX ID | Description |
|---------|---------|-------------|
| MIT License | `MIT` | Permissive, simple |
| Apache License 2.0 | `Apache-2.0` | Permissive, patent grant |
| GNU GPL v3 | `GPL-3.0` | Copyleft, strong |
| BSD 3-Clause | `BSD-3-Clause` | Permissive, with attribution |
| ISC License | `ISC` | Permissive, very simple |

Full list: [SPDX License List](https://spdx.org/licenses/)

## Complete Workflow Examples

### Basic CI Check

```yaml
name: License Check

on:
  push:
    branches: [main]
  pull_request:

jobs:
  license:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Verify license headers
        uses: firestoned/github-actions/security/license-check@v1
        with:
          copyright-holder: 'Erick Bourgeois, firestoned'
          license-id: MIT
```

### Pre-commit Validation

```yaml
name: Pre-commit Checks

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check license headers
        uses: firestoned/github-actions/security/license-check@v1
        with:
          copyright-holder: ${{ vars.COPYRIGHT_HOLDER }}
          license-id: ${{ vars.LICENSE_ID }}

      - name: Run other checks
        run: make lint test
```

### Multi-Project Monorepo

```yaml
name: Monorepo Checks

on: [push, pull_request]

jobs:
  license-check:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        project:
          - { name: 'backend', holder: 'ACME Backend Team', license: 'Apache-2.0' }
          - { name: 'frontend', holder: 'ACME Frontend Team', license: 'MIT' }
    steps:
      - uses: actions/checkout@v4

      - name: Check ${{ matrix.project.name }} license headers
        uses: firestoned/github-actions/security/license-check@v1
        with:
          copyright-holder: ${{ matrix.project.holder }}
          license-id: ${{ matrix.project.license }}
```

### With Auto-Fix on Failure

```yaml
name: License Check with Auto-Fix

on: [pull_request]

jobs:
  license:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check license headers
        id: check
        continue-on-error: true
        uses: firestoned/github-actions/security/license-check@v1
        with:
          copyright-holder: 'Your Name'
          license-id: MIT

      - name: Auto-fix missing headers
        if: failure() && steps.check.outcome == 'failure'
        run: |
          # Add your auto-fix script here
          ./scripts/add-license-headers.sh

      - name: Commit fixes
        if: failure()
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add .
          git commit -m "chore: add missing license headers"
          git push
```

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

### Migration Example

If you were using the old hardcoded action in `bindy`:

```yaml
# Old usage (hardcoded)
- name: Check SPDX headers
  uses: firestoned/bindy/.github/actions/license-check@main
```

Migrate to:

```yaml
# New usage (parameterized)
- name: Check SPDX headers
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Erick Bourgeois, firestoned'
    license-id: MIT
```

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

### 3. Exclude Build Artifacts

Always exclude generated files:

```yaml
- uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Your Name'
    license-id: MIT
    exclude-paths: 'target/,build/,dist/,vendor/,node_modules/,.git/'
```

### 4. Add Pre-commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/usr/bin/env bash
# Copyright (c) 2025 Your Name, your-org
# SPDX-License-Identifier: MIT

set -e

# Run license check locally before committing
docker run --rm -v "$PWD:/workspace" -w /workspace \
  ghcr.io/firestoned/github-actions/security/license-check:latest \
  --copyright-holder "Your Name, your-org" \
  --license-id MIT
```

### 5. Document Required Headers

Add to your CONTRIBUTING.md:

```markdown
## License Headers

All source files must include SPDX license headers:

**Rust files:**
```rust
// Copyright (c) 2025 Your Name, your-org
// SPDX-License-Identifier: MIT
```

**Shell scripts:**
```bash
#!/usr/bin/env bash
# Copyright (c) 2025 Your Name, your-org
# SPDX-License-Identifier: MIT
```

These headers are automatically checked in CI.
```

## Troubleshooting

### Action Fails But Headers Look Correct

**Problem**: Headers are present but check still fails

**Possible causes:**
1. SPDX identifier not in first 10 lines
2. Typo in SPDX identifier
3. Wrong comment style for file type

**Solution**: Verify exact format and placement:

```bash
# Check first 10 lines
head -n 10 src/main.rs

# Search for SPDX identifier
grep -n "SPDX-License-Identifier" src/main.rs
```

### Different Year in Copyright

**Problem**: Copyright shows different year

**Solution**: The action only checks for `SPDX-License-Identifier`, not the year. However, best practice is to update the year:

```rust
// Copyright (c) 2025 Your Name, your-org  ← Update year
// SPDX-License-Identifier: MIT
```

### Excluded Paths Not Working

**Problem**: Files in excluded paths still being checked

**Solution**: Ensure paths include trailing slash:

```yaml
# ✅ CORRECT
exclude-paths: 'target/,build/,.git/'

# ❌ WRONG
exclude-paths: 'target,build,.git'
```

### Multiple Copyright Holders

**Problem**: Project has multiple copyright holders

**Solution**: Use comma-separated list:

```yaml
copyright-holder: 'Author One, Author Two, Organization Name'
```

Or list them separately in the file header:

```rust
// Copyright (c) 2025 Author One
// Copyright (c) 2025 Author Two
// SPDX-License-Identifier: MIT
```

### Non-Standard License

**Problem**: Using a non-standard or custom license

**Solution**: Use the closest SPDX identifier or `LicenseRef-YourLicense`:

```yaml
license-id: LicenseRef-CustomLicense
```

Then document your custom license in your LICENSE file.

## Advanced Usage

### Matrix Build for Multiple Licenses

```yaml
strategy:
  matrix:
    component:
      - { path: 'backend/', holder: 'Backend Team', license: 'Apache-2.0' }
      - { path: 'frontend/', holder: 'Frontend Team', license: 'MIT' }
      - { path: 'shared/', holder: 'Core Team', license: 'BSD-3-Clause' }

steps:
  - uses: actions/checkout@v4

  - name: Check ${{ matrix.component.path }}
    working-directory: ${{ matrix.component.path }}
    uses: firestoned/github-actions/security/license-check@v1
    with:
      copyright-holder: ${{ matrix.component.holder }}
      license-id: ${{ matrix.component.license }}
```

### Generate Report Artifact

```yaml
- name: Check license headers
  id: license-check
  continue-on-error: true
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Your Name'
    license-id: MIT

- name: Generate report
  if: failure()
  run: |
    echo "License check failed at $(date)" > license-report.txt
    echo "Missing headers in:" >> license-report.txt
    # Add list of files without headers

- name: Upload report
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: license-check-report
    path: license-report.txt
```

### Add Headers Automatically

Create a script to add missing headers:

```bash
#!/usr/bin/env bash
# scripts/add-license-headers.sh

COPYRIGHT="Your Name, your-org"
LICENSE="MIT"
YEAR=$(date +%Y)

# Add to Rust files
for file in $(find src -name "*.rs" -not -path "*/target/*"); do
  if ! head -n 10 "$file" | grep -q "SPDX-License-Identifier"; then
    cat > "$file.tmp" <<EOF
// Copyright (c) $YEAR $COPYRIGHT
// SPDX-License-Identifier: $LICENSE

$(cat "$file")
EOF
    mv "$file.tmp" "$file"
  fi
done
```

### Custom Validation Rules

Extend the action with custom validation:

```yaml
- name: Check license headers
  uses: firestoned/github-actions/security/license-check@v1
  with:
    copyright-holder: 'Your Name'
    license-id: MIT

- name: Validate year in copyright
  run: |
    CURRENT_YEAR=$(date +%Y)
    OUTDATED=$(grep -r "Copyright (c)" src/ | grep -v "$CURRENT_YEAR" | wc -l)
    if [ "$OUTDATED" -gt 0 ]; then
      echo "⚠️  Warning: $OUTDATED files have outdated copyright year"
    fi
```

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

### Multiple Licenses

For dual-licensed code:

```rust
// Copyright (c) 2025 Your Name
// SPDX-License-Identifier: MIT OR Apache-2.0
```

### License Exceptions

For licenses with exceptions:

```rust
// Copyright (c) 2025 Your Name
// SPDX-License-Identifier: GPL-3.0-or-later WITH Classpath-exception-2.0
```

## Related Actions

- [verify-signed-commits](../verify-signed-commits/README.md) - Verify commit signatures
- [rust/security-scan](../../rust/security-scan/README.md) - Scan for vulnerabilities
- [security/trivy-scan](../trivy-scan/README.md) - Container security scanning

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Yes (requires bash)
- **Operating Systems**: Linux, macOS, Windows (with bash)
- **File Types**: Rust, Shell, Makefiles, YAML
- **SPDX Version**: Compatible with SPDX 2.x and 3.x

## Resources

- [SPDX License List](https://spdx.org/licenses/)
- [SPDX Specification](https://spdx.github.io/spdx-spec/)
- [Open Source Licensing](https://opensource.org/licenses)
- [Choose a License](https://choosealicense.com/)

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Contributing

Contributions welcome! This action is designed to be:
- License-agnostic (supports all SPDX identifiers)
- Extensible (easy to add new file types)
- Configurable (flexible for different projects)

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.
