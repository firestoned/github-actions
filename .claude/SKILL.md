# Claude Skills Reference

Reusable procedural skills for this GitHub Actions repository. Each skill has a canonical name (kebab-case), trigger conditions, ordered steps, and a verification check. Invoke a skill by name: *"run the action-quality skill"* or *"do a validate-actions"*.

---

## `action-quality`

**When to use:**
- After adding or modifying ANY `action.yml` or `action.yaml` file
- Before committing any action changes
- At the end of EVERY task involving action files (NON-NEGOTIABLE)

**Steps:**
```bash
# 1. Validate YAML syntax for all changed actions
find . -name "action.yml" -o -name "action.yaml" | while read f; do
  python3 -c "import yaml; yaml.safe_load(open('$f'))" && echo "✓ $f"
done

# 2. Check SPDX headers present in first 10 lines
find . -name "action.yml" -o -name "action.yaml" | while read f; do
  head -n 10 "$f" | grep -q "SPDX-License-Identifier:" && echo "✓ SPDX: $f" || echo "✗ MISSING SPDX: $f"
done

# 3. Check branding is present
find . -name "action.yml" -o -name "action.yaml" | while read f; do
  grep -q "branding:" "$f" && echo "✓ branding: $f" || echo "✗ MISSING branding: $f"
done

# 4. Check bash steps use set -euo pipefail
grep -rn "shell: bash" --include="*.yml" --include="*.yaml" -A 2 | grep -v "set -euo pipefail" || true
```

**Verification:** All YAML valid, all SPDX headers present, all branding configured, all bash steps use `set -euo pipefail`.

---

## `validate-actions`

**When to use:**
- Before submitting any PR with action changes
- After modifying action inputs or outputs

**Steps:**
```bash
# 1. Check all actions have README.md
find . -name "action.yml" -o -name "action.yaml" | while read f; do
  dir=$(dirname "$f")
  [ -f "$dir/README.md" ] && echo "✓ README: $dir" || echo "✗ MISSING README: $dir"
done

# 2. Check README length (must be 300-700 lines per CLAUDE.md)
find . -name "action.yml" -o -name "action.yaml" | while read f; do
  dir=$(dirname "$f")
  lines=$(wc -l < "$dir/README.md" 2>/dev/null || echo 0)
  if [ "$lines" -lt 300 ]; then
    echo "✗ README TOO SHORT ($lines lines): $dir/README.md"
  elif [ "$lines" -gt 700 ]; then
    echo "⚠ README LONG ($lines lines): $dir/README.md"
  else
    echo "✓ README ($lines lines): $dir/README.md"
  fi
done

# 3. Validate inputs/outputs match README documentation
# (manual check: confirm all inputs in action.yml appear in README Inputs table)

# 4. Check SPDX headers
find . -name "action.yml" -o -name "action.yaml" | while read f; do
  head -n 10 "$f" | grep -q "SPDX-License-Identifier:" && echo "✓ $f" || echo "✗ $f"
done
```

**Verification:** All READMEs 300-700 lines, all inputs documented, all SPDX headers present.

---

## `update-pr-tests`

**When to use:**
- After adding new inputs or outputs to any action
- After adding a new action entirely
- After changing action behavior

**Steps:**
1. Open `.github/workflows/pr.yml`
2. Find the test job(s) for the modified action
3. Add or update test cases to cover:
   - All new inputs (with valid and invalid values)
   - All new outputs (verify they contain expected values)
   - Negative test cases (verify failure when inputs are invalid)
4. Ensure test job uses the actual action (`uses: ./path/to/action`) not manual commands
5. Add output verification steps after action steps

**Verification:** Every input has at least one test case; every output is verified; negative tests pass.

---

## `update-changelog`

**When to use:**
- After ANY change to action files, documentation, or tests

**Steps:**

Open `CHANGELOG.md` (project root) and prepend an entry in this exact format:

```markdown
## [YYYY-MM-DD] - Brief Title

### Changed
- `path/to/file`: Description of the change

### Why
Brief explanation of the reason.
```

Also update `.claude/CHANGELOG.md` (internal) with:

```markdown
## [YYYY-MM-DD HH:MM] - Brief Title

**Author:** <Name of requester>

### Changed
- `path/to/file`: Description of the change

### Why
Brief explanation.

### Impact
- [ ] Breaking change (input/output renamed or removed)
- [ ] New feature (new input/output added)
- [ ] Bug fix
- [ ] Documentation only
```

**Verification:** Entry present in both CHANGELOG files, dated correctly.

---

## `pre-commit-checklist`

**When to use:**
- Before committing any change (mandatory gate)

**Checklist:**

### If ANY `action.yml`/`action.yaml` was modified:
- [ ] SPDX header present in first 10 lines
- [ ] Branding section present (icon + color)
- [ ] All inputs have descriptions and sensible defaults
- [ ] All outputs have descriptions and `value:` references
- [ ] All bash steps use `set -euo pipefail`
- [ ] All variables quoted: `"${VARIABLE}"`
- [ ] Clear error messages with resolution guidance
- [ ] Step summary written to `$GITHUB_STEP_SUMMARY`
- [ ] No hardcoded values (all configurable via inputs)
- [ ] Internal action references use `./path/to/action` (local) or pinned SHAs

### If README was modified:
- [ ] All inputs in action.yml are documented in README Inputs table
- [ ] All outputs in action.yml are documented in README Outputs table
- [ ] At least 5 usage examples
- [ ] Troubleshooting section present
- [ ] Related Actions section cross-references siblings
- [ ] README is 300-700 lines

### If `.github/workflows/pr.yml` was modified:
- [ ] Tests use actual action (`uses: ./path/to/action`) not manual commands
- [ ] Tests only pass inputs that are defined in the action
- [ ] Negative tests (failure scenarios) present
- [ ] Output values verified

### Always:
- [ ] `CHANGELOG.md` updated
- [ ] No secrets, tokens, or credentials in committed files
- [ ] All action dependencies pinned to versions or SHAs (no `@master`)

**Verification:** Every checked box passes. A task is NOT complete until the full checklist is green.

---

## `add-new-action`

**When to use:**
- When adding a new composite action to this repository

**Steps:**
1. Create directory: `<category>/<action-name>/`
2. Create `action.yml` with:
   - SPDX header (first 2 lines)
   - name, description, author
   - branding (icon + color)
   - inputs (all with descriptions and defaults)
   - outputs (if applicable)
   - `runs.using: composite`
3. Create `README.md` (300-700 lines) with all required sections
4. Add test job to `.github/workflows/pr.yml`
5. Run `validate-actions` skill
6. Run `update-changelog` skill

**Verification:** `validate-actions` skill passes; new action appears in main `README.md` action list.
