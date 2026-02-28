# fuongz/skills

A collection of Claude Code skills, distributed as a plugin marketplace.

## Installation

Add the marketplace to Claude Code:

```shell
/plugin marketplace add fuongz/skills
```

## Skills

### `github-write-pr-message`

Generate a well-structured GitHub Pull Request message from your current git diff and commit history.

**Install:**

```shell
/plugin install github-write-pr-message@fuongz-skills
```

**Usage:**

```shell
/github-write-pr-message
/github-write-pr-message focus on the auth changes
```

**What it does:**

- Reads your branch name, commits, changed files, and diff
- Detects existing PR templates in `.github/`, `docs/`, or repo root — and fills them in
- Falls back to a default structure covering Summary, Changes, Related Issues, Type of Change, Testing, and a Checklist
- Outputs a ready-to-paste PR title and body (compatible with `gh pr create`)

## License

MIT
