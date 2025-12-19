# Release Process

This document explains how to release new versions of the firestoned/github-actions repository.

## Overview

The release process is **event-driven** and automated. When you publish a GitHub Release, the workflow automatically:

1. Validates the version format
2. Validates all action files (syntax, required fields, documentation, licenses)
3. Runs the test suite
4. Creates major and minor version tags (for stable releases)
5. Verifies all tags exist
6. Provides usage instructions

## Release Workflow

### Trigger: `release.published` Event

The [.github/workflows/release.yml](.github/workflows/release.yml) workflow runs automatically when you **publish a release** on GitHub.

### Workflow Jobs

```
extract-version → validate → test → create-tags → post-release
                    ↓
                (all jobs depend on extract-version)
```

1. **extract-version**: Parses the release tag and extracts version information
2. **validate**: Validates all action files, documentation, and licenses
3. **test**: Runs functional tests on key actions
4. **create-tags**: Creates major (`v1`) and minor (`v1.0`) tags for stable releases
5. **post-release**: Verifies tags exist and displays usage instructions

## How to Release

### Step 1: Update CHANGELOG.md

Before creating a release, ensure your [CHANGELOG.md](CHANGELOG.md) has an entry for the version:

```markdown
## [1.0.0] - 2025-12-18

### Added
- New feature descriptions
- Bug fixes
- Breaking changes (if any)
```

### Step 2: Create and Publish a GitHub Release

#### Option A: Using GitHub Web UI

1. Go to **Releases** → **Draft a new release**
2. Click **Choose a tag**
3. Type the version (e.g., `v1.0.0`) and click **Create new tag**
4. Fill in:
   - **Release title**: `v1.0.0` (or descriptive title)
   - **Description**: Copy from CHANGELOG.md or write release notes
   - **Set as a pre-release**: Check this for beta/RC versions
5. Click **Publish release**

#### Option B: Using GitHub CLI

```bash
# Stable release
gh release create v1.0.0 \
  --title "v1.0.0 - Initial Stable Release" \
  --notes "$(sed -n '/## \[1.0.0\]/,/## \[/p' CHANGELOG.md | sed '$d')"

# Pre-release
gh release create v1.1.0-beta.1 \
  --title "v1.1.0-beta.1" \
  --notes "Beta release for testing" \
  --prerelease
```

### Step 3: Workflow Runs Automatically

Once you publish the release, the workflow will:

- ✅ Validate the release tag format (must be semver: `vX.Y.Z` or `vX.Y.Z-prerelease`)
- ✅ Run all validation checks
- ✅ Run the test suite
- ✅ Create moving tags (only for stable releases):
  - `v1.0` → points to latest `v1.0.x`
  - `v1` → points to latest `v1.x.x`
- ✅ Verify everything is ready

### Step 4: Monitor Workflow

1. Go to **Actions** → **Release GitHub Actions**
2. Watch the workflow run
3. Check the summary for usage instructions

## Version Tags Explained

### Exact Version Tag (Created by Release)

When you create a release with tag `v1.0.0`, GitHub creates this tag automatically.

**Example**: `v1.0.0`
- Never moves
- Always points to the same commit
- Use for reproducible builds

### Moving Tags (Created by Workflow)

For **stable releases only** (not pre-releases), the workflow creates/updates:

**Minor Tag**: `v1.0`
- Points to the latest `v1.0.x` release
- Updates when you release `v1.0.1`, `v1.0.2`, etc.
- Users get patch updates automatically

**Major Tag**: `v1`
- Points to the latest `v1.x.x` release
- Updates when you release `v1.1.0`, `v1.2.0`, etc.
- Users get minor and patch updates automatically

### Tag Behavior for Pre-releases

Pre-releases (e.g., `v1.1.0-beta.1`) **DO NOT** update moving tags.

**Why?** Users who pin to `@v1` should only get stable releases, not beta versions.

## Release Types

### Stable Release (v1.0.0, v1.1.0, v2.0.0)

```bash
gh release create v1.0.0 --title "v1.0.0" --notes "Release notes here"
```

**Result**:
- Creates tag `v1.0.0` ✅
- Updates tag `v1.0` → `v1.0.0` ✅
- Updates tag `v1` → `v1.0.0` ✅

**Users can reference**:
```yaml
uses: firestoned/github-actions/rust/cache-cargo@v1       # Recommended
uses: firestoned/github-actions/rust/cache-cargo@v1.0     # Conservative
uses: firestoned/github-actions/rust/cache-cargo@v1.0.0   # Exact
```

### Patch Release (v1.0.1)

```bash
gh release create v1.0.1 --title "v1.0.1" --notes "Bug fixes"
```

**Result**:
- Creates tag `v1.0.1` ✅
- Updates tag `v1.0` → `v1.0.1` ✅ (moves forward)
- Updates tag `v1` → `v1.0.1` ✅ (moves forward)

### Minor Release (v1.1.0)

```bash
gh release create v1.1.0 --title "v1.1.0" --notes "New features"
```

**Result**:
- Creates tag `v1.1.0` ✅
- Creates tag `v1.1` → `v1.1.0` ✅ (new minor tag)
- Updates tag `v1` → `v1.1.0` ✅ (moves forward)

### Pre-release (v1.1.0-beta.1)

```bash
gh release create v1.1.0-beta.1 \
  --title "v1.1.0-beta.1" \
  --notes "Beta release" \
  --prerelease
```

**Result**:
- Creates tag `v1.1.0-beta.1` ✅
- **DOES NOT** update `v1.1` or `v1` ⚠️

**Users must use exact version**:
```yaml
uses: firestoned/github-actions/rust/cache-cargo@v1.1.0-beta.1
```

## Semantic Versioning

This repository follows [Semantic Versioning 2.0.0](https://semver.org/):

### Major Version (v1.0.0 → v2.0.0)

**Breaking changes** to action inputs/outputs.

**Example**:
- Removing an input parameter
- Changing output format
- Renaming an action

**Users**: Must update their workflows manually.

### Minor Version (v1.0.0 → v1.1.0)

**New features**, backward compatible.

**Example**:
- Adding a new optional input
- Adding a new output
- Adding a new action

**Users**: Get updates automatically if using `@v1`.

### Patch Version (v1.0.0 → v1.0.1)

**Bug fixes**, backward compatible.

**Example**:
- Fixing a bug
- Updating documentation
- Security patches

**Users**: Get updates automatically if using `@v1` or `@v1.0`.

## Validation Checks

The workflow runs these validations before creating tags:

### 1. Version Format

```bash
# Valid
v1.0.0
v1.0.0-beta.1
v2.0.0-rc.1

# Invalid
1.0.0           # Missing 'v' prefix
v1.0            # Missing patch version
v1.0.0.1        # Too many segments
```

### 2. Action Files

- All `action.yml` files must be valid YAML
- Required fields: `name`, `description`, `runs.using`
- Must have SPDX license headers

### 3. Documentation

- Every action must have a `README.md`
- Must include usage examples

### 4. License

- Repository must have `LICENSE` file
- All action files must have SPDX headers

### 5. Functional Tests

- Tests run for key actions (cache-cargo, extract-version, license-check, setup-docker)
- Docker Buildx verification

## Troubleshooting

### Workflow Failed During Validation

**Problem**: Action files have syntax errors or missing fields.

**Solution**:
1. Check the workflow logs for specific errors
2. Fix the issues in your code
3. Delete the release (if it was published)
4. Create a new release with a new tag

### Tags Not Created

**Problem**: Moving tags (`v1`, `v1.0`) weren't created.

**Possible causes**:
1. **Pre-release**: Moving tags are intentionally skipped for pre-releases
2. **Workflow failed**: Check the workflow logs
3. **Permissions**: Workflow needs `contents: write` permission (already configured)

**Solution**: Check workflow logs in the Actions tab.

### Wrong Version Tag Format

**Problem**: Workflow failed because tag doesn't match semver format.

**Solution**:
1. Delete the release
2. Create a new release with correct format: `vX.Y.Z`

### Need to Re-run Workflow

**Problem**: Workflow failed but you've fixed the issue.

**Solution**:
You can re-run the workflow from the Actions tab, but **only if** the release tag hasn't changed. If you need to change the tag:
1. Delete the release
2. Delete the tag: `git push origin :refs/tags/vX.Y.Z`
3. Create a new release with the corrected tag

## Best Practices

### 1. Always Update CHANGELOG.md First

Before creating a release, document the changes in CHANGELOG.md.

### 2. Test Before Releasing

Ensure all tests pass locally:
```bash
# Run validation manually
find . -name "action.yml" -o -name "action.yaml" | while read action; do
  python3 -c "import yaml; yaml.safe_load(open('$action'))"
done
```

### 3. Use Pre-releases for Testing

For beta/RC versions, mark the release as a pre-release:
```bash
gh release create v1.1.0-beta.1 --prerelease
```

### 4. Don't Skip Versions

Release versions in order: `v1.0.0` → `v1.0.1` → `v1.1.0` → `v2.0.0`

Don't skip from `v1.0.0` to `v1.2.0`.

### 5. Communicate Breaking Changes

For major version bumps, clearly document:
- What changed
- Migration guide
- Timeline for deprecation (if applicable)

## Example Release Workflow

### Releasing v1.0.1 (Patch)

```bash
# 1. Update CHANGELOG.md
cat >> CHANGELOG.md <<EOF

## [1.0.1] - $(date +%Y-%m-%d)

### Fixed
- Fixed bug in cache-cargo action
- Updated documentation
EOF

# 2. Commit changes
git add CHANGELOG.md
git commit -m "Prepare v1.0.1 release"
git push origin main

# 3. Create and publish release
gh release create v1.0.1 \
  --title "v1.0.1 - Bug Fixes" \
  --notes "$(sed -n '/## \[1.0.1\]/,/## \[/p' CHANGELOG.md | sed '$d')"

# 4. Watch workflow run
gh run watch
```

**Result**: Tags `v1.0.1`, `v1.0`, and `v1` are created/updated.

### Releasing v1.1.0-beta.1 (Pre-release)

```bash
# 1. No CHANGELOG update needed for beta

# 2. Create and publish pre-release
gh release create v1.1.0-beta.1 \
  --title "v1.1.0-beta.1 - Beta Release" \
  --notes "Testing new features before stable release" \
  --prerelease

# 3. Watch workflow run
gh run watch
```

**Result**: Only tag `v1.1.0-beta.1` is created. Moving tags are NOT updated.

## Summary

✅ **Create a GitHub Release** → Workflow runs automatically
✅ **Workflow validates** → Ensures quality
✅ **Workflow creates tags** → Users get updates (for stable releases)
✅ **All automated** → No manual tag management needed

---

For questions or issues, see [GitHub Issues](https://github.com/firestoned/github-actions/issues).
