## [2026-10-13 21:10] - rust/cache-cargo: rebuild on Swatinem/rust-cache with per-job keys

**Author:** Erick Bourgeois

### Changed
- `rust/cache-cargo/action.yml`: wraps `Swatinem/rust-cache` v2.9.1 (pinned SHA,
  same as `rust/setup-rust-build`); new optional inputs `key`, `shared-key`,
  `save-if`, `cache-on-failure`, `workspaces`, `prefix-key`; `cache-hit` output;
  toolchain precondition check; step summary.
- `rust/cache-cargo/README.md`: rewritten for the new keying, with a migration
  section and the measurements that motivated it.
- `.github/workflows/pr.yml`: real tests for `rust/cache-cargo` (matrix over
  inputs, output check, build after restore, negative test without a toolchain).
- `CHANGELOG.md`: Unreleased entry.

### Why
One shared `target/` key per OS and lockfile made unrelated jobs overwrite each
other's caches and grew without bound. Measured in finos/5-spot: a 4.4 GB entry,
2m42s to restore, 38 crates recompiled anyway, and the repository at 9.85 GB of
the 10 GB quota.

### Impact
- [ ] Breaking change (no required inputs; first run after upgrade is cold)
- [x] New feature (inputs and output)
- [x] Bug fix (cache keying, unbounded growth, vacuous test)
- [ ] Documentation only

---

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
