# Verify Signed Commits

A composite GitHub Action that verifies all commits in pull requests, pushes, or releases are cryptographically signed with GPG or SSH keys using the GitHub API.

## Features

- Verifies commit signatures using GitHub's API
- Supports multiple verification modes (PR, push, release)
- Detailed reporting of unsigned commits
- Fails workflows with unsigned commits
- Uses GitHub's "Verified" badge system
- Handles new branches and force pushes
- Clear error messages with setup instructions
- No local GPG/SSH configuration needed

## Usage

### Pull Request Verification

```yaml
name: PR Checks

on:
  pull_request:

jobs:
  verify-signatures:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Fetch all history

      - name: Verify signed commits
        uses: your-org/github-actions/security/verify-signed-commits@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          base-ref: origin/${{ github.base_ref }}
          verify-mode: pr
```

### Push Verification

```yaml
name: Push Checks

on:
  push:
    branches: [main, develop]

jobs:
  verify-signatures:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Verify signed commits
        uses: your-org/github-actions/security/verify-signed-commits@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          verify-mode: push
```

### Release Verification

```yaml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  verify-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Verify release commit is signed
        uses: your-org/github-actions/security/verify-signed-commits@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          verify-mode: release

      - name: Create release
        run: gh release create ${{ github.ref_name }}
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Complete CI Workflow

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Verify signed commits
        uses: your-org/github-actions/security/verify-signed-commits@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          base-ref: ${{ github.event_name == 'pull_request' && format('origin/{0}', github.base_ref) || '' }}
          verify-mode: ${{ github.event_name == 'pull_request' && 'pr' || 'push' }}
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `github-token` | GitHub token for API access (use `${{ secrets.GITHUB_TOKEN }}`) | Yes | N/A |
| `base-ref` | Base reference for comparison (e.g., `origin/main` or commit SHA) | No | `''` |
| `verify-mode` | Verification mode: `pr`, `push`, or `release` | Yes | N/A |

## Verification Modes

### PR Mode (`verify-mode: pr`)

Verifies all commits in a pull request:
- Compares `base-ref..HEAD`
- Requires `base-ref` to be set (usually `origin/${{ github.base_ref }}`)
- Checks all commits added in the PR

Example:
```yaml
verify-mode: pr
base-ref: origin/main
```

### Push Mode (`verify-mode: push`)

Verifies commits in a push:
- For existing branches: Compares `github.event.before..HEAD`
- For new branches: Checks only the latest commit
- Automatically handles new branch detection

Example:
```yaml
verify-mode: push
# base-ref not needed
```

### Release Mode (`verify-mode: release`)

Verifies only the release commit:
- Checks `github.sha`
- Used for tag-based releases
- Ensures release commits are signed

Example:
```yaml
verify-mode: release
# base-ref not needed
```

## How It Works

### Verification Process

1. **Determine commits**: Based on verification mode
2. **List commits**: Shows all commits being verified
3. **Query GitHub API**: For each commit, check verification status
4. **Collect results**: Track unsigned commits
5. **Report**: Display results and fail if any unsigned

### GitHub API Verification

The action uses GitHub's commit verification endpoint:
```bash
gh api repos/{owner}/{repo}/commits/{sha} --jq '.commit.verification.verified'
```

Returns:
- `true`: Commit is signed and verified
- `false`: Commit is not signed or verification failed

### Signature Types Supported

GitHub verifies:
- **GPG signatures**: Traditional GPG signing
- **SSH signatures**: Modern SSH key signing
- **S/MIME signatures**: Email-based signing

All show "Verified" badge on GitHub when valid.

## Required Permissions

```yaml
permissions:
  contents: read  # Required to read repository commits
```

The default `GITHUB_TOKEN` has sufficient permissions.

## Best Practices

### 1. Enforce on All PRs

Make signature verification required:

```yaml
# .github/workflows/pr-checks.yml
name: PR Checks

on:
  pull_request:

jobs:
  verify-signatures:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Important!

      - uses: your-org/github-actions/security/verify-signed-commits@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          base-ref: origin/${{ github.base_ref }}
          verify-mode: pr
```

Then add to branch protection rules:
- Required status check: `verify-signatures`

### 2. Fetch Full History

Always use `fetch-depth: 0` for accurate verification:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # Required for git log
```

### 3. Verify Before Merge

Add as a required check in branch protection:

Settings → Branches → Branch protection rules:
- ✓ Require status checks to pass
- ✓ `verify-signatures`

### 4. Document Requirements

Add to `CONTRIBUTING.md`:

```markdown
## Commit Signing

All commits must be signed with GPG or SSH keys.

### Setup GPG Signing

1. Generate GPG key:
   \```bash
   gpg --full-generate-key
   \```

2. Get key ID:
   \```bash
   gpg --list-secret-keys --keyid-format LONG
   \```

3. Configure git:
   \```bash
   git config --global user.signingkey YOUR_KEY_ID
   git config --global commit.gpgsign true
   \```

4. Add to GitHub:
   - Settings → SSH and GPG keys → New GPG key
   - Paste: `gpg --armor --export YOUR_KEY_ID`

### Setup SSH Signing

1. Generate SSH key (if needed):
   \```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   \```

2. Configure git:
   \```bash
   git config --global gpg.format ssh
   git config --global user.signingkey ~/.ssh/id_ed25519.pub
   git config --global commit.gpgsign true
   \```

3. Add to GitHub:
   - Settings → SSH and GPG keys → New SSH key
   - Key type: Signing Key
```

### 5. Handle Force Pushes

For workflows on protected branches, force pushes create new history:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
    ref: ${{ github.event.after }}  # Handle force push
```

## Troubleshooting

### "No commits to verify"

**Problem**: Action completes but says no commits to verify

**Solution**: Check `fetch-depth` and `base-ref`:
```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # Must fetch history

- uses: your-org/github-actions/security/verify-signed-commits@v1
  with:
    base-ref: origin/${{ github.base_ref }}  # Must specify for PRs
    verify-mode: pr
```

### Unsigned Commits Found

**Problem**: Verification fails with unsigned commits

**Solution**: Developers must sign commits:

1. **Check which commits are unsigned**:
   ```bash
   git log --show-signature
   ```

2. **Amend last commit with signature**:
   ```bash
   git commit --amend --no-edit -S
   git push --force-with-lease
   ```

3. **Rebase and sign all commits**:
   ```bash
   git rebase --exec 'git commit --amend --no-edit -n -S' -i main
   git push --force-with-lease
   ```

### GitHub API Rate Limit

**Problem**: API requests fail with rate limit error

**Solution**: Use `GITHUB_TOKEN` (not personal access token):
```yaml
github-token: ${{ secrets.GITHUB_TOKEN }}
```

Limits:
- `GITHUB_TOKEN`: 1000 requests/hour
- Personal token: 5000 requests/hour

### Verified Locally but Not on GitHub

**Problem**: Commit shows signed locally but not verified on GitHub

**Solution**: Ensure GPG/SSH key is added to GitHub:

1. Check key is on GitHub:
   - Settings → SSH and GPG keys

2. Verify key matches:
   ```bash
   # For GPG
   gpg --list-keys

   # For SSH
   cat ~/.ssh/id_ed25519.pub
   ```

3. Re-upload key if needed

## Advanced Usage

### Dynamic Mode Selection

Choose mode based on event type:

```yaml
- name: Verify commits
  uses: your-org/github-actions/security/verify-signed-commits@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    base-ref: ${{ github.event_name == 'pull_request' && format('origin/{0}', github.base_ref) || '' }}
    verify-mode: ${{ github.event_name == 'pull_request' && 'pr' || (github.ref_type == 'tag' && 'release' || 'push') }}
```

### Custom Error Handling

To warn instead of fail:

```yaml
- name: Verify commits (warn only)
  uses: your-org/github-actions/security/verify-signed-commits@v1
  continue-on-error: true
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    verify-mode: pr
```

### Notify on Failure

Send notifications for unsigned commits:

```yaml
- name: Verify commits
  id: verify
  uses: your-org/github-actions/security/verify-signed-commits@v1
  continue-on-error: true
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    verify-mode: pr

- name: Notify on failure
  if: steps.verify.outcome == 'failure'
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Unsigned commits found in PR #${{ github.event.pull_request.number }}"
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Verify Specific Commit Range

For custom ranges:

```yaml
- name: Verify specific range
  uses: your-org/github-actions/security/verify-signed-commits@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    base-ref: abc123  # Specific commit SHA
    verify-mode: pr
```

## Understanding Commit Signatures

### What is Verified?

When GitHub shows "Verified":
1. Signature is cryptographically valid
2. Signing key is associated with a GitHub account
3. Committer email matches GitHub account
4. Key was not revoked at time of commit

### Signature Methods Compared

| Method | Key Type | Setup Complexity | Security | GitHub Support |
|--------|----------|------------------|----------|----------------|
| GPG | RSA/EdDSA | Medium | High | ✓ Full |
| SSH | Ed25519/RSA | Low | High | ✓ Full |
| S/MIME | X.509 | High | High | ✓ Full |

### Viewing Signatures Locally

```bash
# View signature for a commit
git show --show-signature <commit>

# View signatures for all commits
git log --show-signature

# Verify signature
git verify-commit <commit>
```

## Compliance and Auditing

### Regulatory Requirements

Commit signing helps meet:
- **SOC 2**: Code integrity and change management
- **ISO 27001**: Secure development lifecycle
- **PCI DSS**: Code change tracking and approval
- **FedRAMP**: Software supply chain security

### Audit Trail

Signed commits provide:
- **Attribution**: Who made the change
- **Integrity**: Change was not tampered with
- **Non-repudiation**: Cannot deny authorship
- **Timestamp**: When change was made

### Reporting

Generate signed commit reports:

```bash
# List all unsigned commits
git log --pretty=format:"%H %an %s" --no-show-signature | \
  while read sha author subject; do
    if ! git verify-commit $sha 2>/dev/null; then
      echo "UNSIGNED: $sha - $author - $subject"
    fi
  done
```

## Performance Tips

### 1. Limit Commit Range

For large PRs, verification time scales with commit count. Consider squashing commits.

### 2. Cache Checkout

For multiple jobs:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- uses: actions/cache@v4
  with:
    path: .git
    key: git-${{ github.sha }}
```

### 3. Parallel Verification

For very large repos, split verification:

```yaml
strategy:
  matrix:
    shard: [1, 2, 3, 4]
steps:
  - name: Verify shard
    run: |
      # Custom logic to verify 1/4 of commits
```

## Related Actions

- [security/cosign-sign](../cosign-sign/README.md) - Sign artifacts and images
- [security/trivy-scan](../trivy-scan/README.md) - Security scanning
- [rust/security-scan](../../rust/security-scan/README.md) - Dependency scanning

## License

MIT License - Copyright (c) 2025 Erick Bourgeois, firestoned

## Compatibility

- **GitHub Actions**: All GitHub-hosted runners
- **Self-hosted runners**: Requires `git` and `gh` CLI
- **GitHub API**: Uses GitHub REST API v3
- **Git versions**: Tested with Git 2.30+

## Resources

- [GitHub Commit Signature Verification](https://docs.github.com/en/authentication/managing-commit-signature-verification)
- [GPG Documentation](https://gnupg.org/documentation/)
- [SSH Signature Support](https://github.blog/2022-08-23-ssh-commit-verification-now-supported/)
- [Git Signature Documentation](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)

## Examples

### Branch Protection with Signature Verification

```yaml
# .github/workflows/branch-protection.yml
name: Branch Protection

on:
  pull_request:
    branches: [main, release/*]

jobs:
  verify-signatures:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Verify all commits are signed
        uses: your-org/github-actions/security/verify-signed-commits@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          base-ref: origin/${{ github.base_ref }}
          verify-mode: pr

  code-review:
    needs: verify-signatures
    runs-on: ubuntu-latest
    steps:
      - name: Run code review
        run: echo "Signatures verified, proceeding with review"
```

### Multi-Environment Verification

```yaml
jobs:
  verify-dev:
    if: github.base_ref == 'develop'
    steps:
      - uses: your-org/github-actions/security/verify-signed-commits@v1
        continue-on-error: true  # Warn only for dev
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          verify-mode: pr

  verify-prod:
    if: github.base_ref == 'main'
    steps:
      - uses: your-org/github-actions/security/verify-signed-commits@v1
        # Fail for production
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          verify-mode: pr
```
