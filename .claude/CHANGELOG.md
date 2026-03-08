## [2026-03-19 00:00] - Initial skill setup and action upgrades

**Author:** Erick Bourgeois

### Changed
- `.claude/SKILL.md`: Created github-actions-specific skills (action-quality, validate-actions, update-pr-tests, pre-commit-checklist, add-new-action)
- `.claude/rules/rust-style.md`: Copied from bindy project (Rust style guide)
- `.claude/rules/testing.md`: Copied from bindy project (testing standards)
- `security/trivy-scan/action.yaml`: Fixed unpinned @master dependency, added format/severity/exit-code/output inputs
- `rust/package-crate/action.yaml`: Added branding, set -euo pipefail, step summary, outputs
- `rust/publish-crate/action.yaml`: Added branding, set -euo pipefail, step summary, fixed cargo login
- `rust/build-binary/action.yaml`: Added outputs (binary-path), step summary, macOS target support
- `rust/package-crate/README.md`: Expanded from 134 to 300+ lines
- `rust/publish-crate/README.md`: Expanded from 180 to 300+ lines
- `rust/cache-cargo/README.md`: Expanded from 243 to 300+ lines
- `.github/workflows/pr.yml`: Fixed trivy-scan tests to use valid inputs only

### Why
Initial skill framework setup and compliance pass to meet CLAUDE.md standards — all actions must have branding, set -euo pipefail, GITHUB_STEP_SUMMARY, and READMEs of 300-700 lines. Trivy-scan had a critical unpinned @master dependency.

### Impact
- [ ] Breaking change
- [x] New feature (new inputs added to trivy-scan, build-binary)
- [x] Bug fix (trivy-scan pinned, cargo login fixed)
- [x] Documentation only (README expansions)
