# fuongz/skills

Claude Code skill plugins by fuongz.

## Installation

Add the marketplace:

```shell
/plugin marketplace add fuongz/skills
```

Then install individual skills:

```shell
/plugin install <skill-name>@fuongz-skills
```

---

## Skills

| Skill | Description |
|-------|-------------|
| [`init-stack`](#init-stack) | Scaffold a full-stack app with TanStack Start + Cloudflare + shadcn/ui |
| [`github-write-pr-message`](#github-write-pr-message) | Generate a GitHub PR message from your git diff |

---

### `init-stack`

Scaffold a production-ready full-stack app from scratch.

**Stack:** TanStack Start · Cloudflare Workers · shadcn/ui · Tailwind CSS v4 · Biome · Hugeicons · bun

**Install:**

```shell
/plugin install init-stack@fuongz-skills
```

**Usage:**

```shell
/init-stack my-app
/init-stack ~/projects/my-app
```

**What it does:**

- Initialises a bun project and installs all dependencies
- Writes `tsconfig.json`, `vite.config.ts`, and `wrangler.jsonc` with correct settings
- Sets up Biome for linting and formatting (`lint` / `lint:fix` scripts)
- Scaffolds `src/router.tsx`, `src/routes/__root.tsx`, and `src/routes/index.tsx`
- Runs the dev server once to generate `routeTree.gen.ts`
- Initialises shadcn/ui with the `base-nova` preset (OKLCH colours, Inter font, Hugeicons)
- Installs Hugeicons and runs a final typecheck

---

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
- Detects existing PR templates in `.github/`, `docs/`, or repo root and fills them in
- Falls back to a default structure: Summary, Changes, Related Issues, Type of Change, Testing, Checklist
- Outputs a ready-to-paste PR title and body (compatible with `gh pr create`)

---

## License

MIT
