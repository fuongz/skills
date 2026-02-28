---
name: Write_PR_Message
description: Generate a GitHub Pull Request message following GitHub's PR template guidelines. Use when the user wants to create a PR, write a pull request description, or asks to summarize changes for a PR.
argument-hint: "[context or focus area]"
---

# GitHub Pull Request Message Generator

Generate a well-structured GitHub Pull Request message following GitHub's official PR template guidelines.

**Additional context (optional):** $ARGUMENTS

## Git Context

- Branch: !`git branch --show-current`
- Commits: !`git log main..HEAD --oneline 2>/dev/null || git log master..HEAD --oneline 2>/dev/null || git log --oneline -10`
- Changed files: !`git diff --name-status main..HEAD 2>/dev/null || git diff --name-status master..HEAD 2>/dev/null || git diff --name-status HEAD~1..HEAD`
- Diff: !`git diff HEAD`

## Workflow

### 1. Detect PR Template

Check for existing PR templates (in priority order):

1. `.github/pull_request_template.md`
2. `.github/PULL_REQUEST_TEMPLATE/*.md`
3. `docs/pull_request_template.md`
4. `pull_request_template.md`

If found, use it as the structure and fill all sections. If not found, use the **Default PR Structure** below.

### 2. Analyze the Changes

- **What changed**: Files modified, added, or deleted
- **Why it changed**: Purpose from commit messages, branch name, and diff
- **Impact**: Areas of the codebase affected
- **Related issues**: Extract issue numbers from branch name (e.g. `fix/123-bug`) or commit messages

### 3. Generate the PR Message

#### Default PR Structure

```markdown
## Summary

<!-- Brief description of what this PR does and why -->

## Changes

-
-

## Related Issues

Closes #<issue-number>

## Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Refactoring
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Test addition or update

## Testing

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manually tested

## Screenshots / Demo

## Checklist

- [ ] Self-review completed
- [ ] Code follows project style guidelines
- [ ] No new warnings introduced
- [ ] Tests added where applicable
- [ ] Dependent changes merged and published
```

### 4. Output Format

Present in this order:

1. **PR Title** — Imperative mood, 50–72 chars, conventional commit prefix if used (`feat:`, `fix:`, `chore:`, etc.)
2. **PR Body** — Full markdown ready to paste into GitHub or use with `gh pr create`
3. **Reviewer suggestions** — (Optional) `@mentions` based on changed files

### Title Guidelines

**Good:**
- `fix: resolve null pointer in user authentication`
- `feat: add dark mode support to settings panel`

**Bad:** `Fix bug`, `Changes`, `WIP`

### GitHub Rules

- Use closing keywords (`Closes`, `Fixes`, `Resolves`) + `#issue-number` to auto-close issues on merge
- Fill all template sections — do not omit any
- If multiple templates exist in `.github/PULL_REQUEST_TEMPLATE/`, present as options and use the most relevant

---

Output the PR title and body in a code block for easy copy-paste or use with `gh pr create`.
