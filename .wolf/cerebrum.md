---
description: learned preferences, project conventions, and Do-Not-Repeat rules
budget_tokens: 2000
---
# Cerebrum

> OpenWolf's learning memory. Updated automatically as the AI learns from interactions.
> Do not edit manually unless correcting an error.
> Last updated: 2026-10-03

## User Preferences

<!-- How the user likes things done. Code style, tools, patterns, communication. -->

## Key Learnings

- **Project:** github-actions
- **Description:** **Reusable composite GitHub Actions for CI/CD pipelines**

## Do-Not-Repeat

<!-- Mistakes made and corrected. Each entry prevents the same mistake recurring. -->
<!-- Format: [YYYY-MM-DD] Description of what went wrong and what to do instead. -->
- [2026-10-03] `openwolf init` wrote absolute home-dir paths into `.claude/settings.json` hook commands. Tracked OpenWolf/opencode/Claude files must hold no personal data or home paths: use `$CLAUDE_PROJECT_DIR` in hooks and run the sweep in `.claude/rules/no-real-infrastructure.md` before committing.

## Decision Log

<!-- Significant technical decisions with rationale. Why X was chosen over Y. -->
