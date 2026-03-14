---
description: >
  Scaffold a new full-stack project with TanStack Start + Cloudflare Workers + shadcn/ui +
  Tailwind CSS v4 + Hugeicons + bun. Use this skill whenever the user says "init project",
  "bootstrap a new app", "set up the stack", "create a new project", "start a new app",
  or any variation of initializing a fresh codebase with this tech stack. Also trigger when
  the user asks to recreate the boilerplate, start from scratch, or set up a new repo with
  TanStack, Cloudflare Workers, or shadcn. Do not wait for the user to list every technology
  — if it sounds like they want a new project and this stack is in scope, use this skill.
argument-hint: "[project name or directory]"
---

# Init Stack

Bootstrap a production-ready full-stack app: **TanStack Start** · **Cloudflare Workers** · **shadcn/ui** · **Tailwind CSS v4** · **Hugeicons** · **bun**

**Project name/path (if provided):** $ARGUMENTS

---

## Before You Start

If `$ARGUMENTS` is empty, ask the user: "What's the project name and where should I create it?" Otherwise, use the provided name/path directly. The project name goes into `wrangler.jsonc` and `package.json`.

cd into the target directory before running any commands.

---

## Step 1 — Init bun project

```bash
bun init -y
rm -f index.ts README.md   # clean up bun's default files
```

---

## Step 2 — Install dependencies

```bash
# Runtime deps
bun add @tanstack/react-start @tanstack/react-router react react-dom tailwindcss @tailwindcss/vite

# Dev deps
bun add -D vite @vitejs/plugin-react vite-tsconfig-paths @types/react @types/react-dom @cloudflare/vite-plugin wrangler typescript @biomejs/biome
```

---

## Step 3 — Init Biome

```bash
bunx biome init
```

Then overwrite `biome.json` with the following (uses local `$schema` from node_modules):

```json
{
  "$schema": "./node_modules/@biomejs/biome/configuration_schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "files": {
    "includes": ["**", "!**/src/routeTree.gen.ts"]
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "tab"
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "double"
    }
  },
  "css": {
    "parser": {
      "tailwindDirectives": true
    }
  },
  "assist": {
    "enabled": true,
    "actions": {
      "source": {
        "organizeImports": "on"
      }
    }
  }
}
```

---

## Step 4 — Write `tsconfig.json`

> **Why these settings matter:**
> - `moduleResolution: "Bundler"` is required for Vite's import resolution to work correctly.
> - `verbatimModuleSyntax` must be **omitted** — TanStack Start docs explicitly warn it causes server bundle leakage into client bundles.
> - The `@/*` alias maps to `src/*` for clean imports throughout the project.

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "moduleResolution": "Bundler",
    "module": "ESNext",
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "strictNullChecks": true,
    "strict": true,
    "noEmit": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src", "vite.config.ts"]
}
```

---

## Step 5 — Write `vite.config.ts`

> **Plugin order is critical.** `cloudflare()` must come before `tanstackStart()`, and `viteReact()` must come after `tanstackStart()`. The `cloudflare` plugin targets the SSR environment so the Workers bundle is created correctly.

```ts
import { defineConfig } from 'vite'
import { cloudflare } from '@cloudflare/vite-plugin'
import { tanstackStart } from '@tanstack/react-start/plugin/vite'
import viteReact from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'
import tsConfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [
    tsConfigPaths(),
    tailwindcss(),
    cloudflare({ viteEnvironment: { name: 'ssr' } }),
    tanstackStart(),
    // React plugin must come AFTER TanStack Start plugin
    viteReact(),
  ],
})
```

---

## Step 6 — Write `wrangler.jsonc`

Replace `<project-name>` with the actual project name.

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "<project-name>",
  "compatibility_date": "2025-09-02",
  "compatibility_flags": ["nodejs_compat"],
  "main": "@tanstack/react-start/server-entry",
  "assets": {
    "directory": ".output/static",
    "binding": "ASSETS"
  }
}
```

---

## Step 7 — Update `package.json`

Replace the `bun init` defaults with proper scripts. Keep all dependencies that were installed.

```json
{
  "name": "<project-name>",
  "type": "module",
  "private": true,
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview",
    "deploy": "bun run build && wrangler deploy",
    "typecheck": "tsc --noEmit",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "cf-typegen": "wrangler types"
  }
}
```

---

## Step 8 — Scaffold source files

Create the following files exactly as shown.

### `src/styles/globals.css`
```css
@import "tailwindcss";
```
*(shadcn init will expand this file with the full theme in Step 9)*

### `src/router.tsx`
```tsx
import { createRouter as createTanStackRouter } from '@tanstack/react-router'
import { routeTree } from './routeTree.gen'

export function createRouter() {
  const router = createTanStackRouter({
    routeTree,
    scrollRestoration: true,
  })
  return router
}

// Required by TanStack Start v1.166+ — the server calls getRouter() at runtime
export async function getRouter() {
  return createRouter()
}

declare module '@tanstack/react-router' {
  interface Register {
    router: ReturnType<typeof createRouter>
  }
}
```

### `src/routes/__root.tsx`
```tsx
/// <reference types="vite/client" />
import type { ReactNode } from 'react'
import '../styles/globals.css'
import {
  Outlet,
  createRootRoute,
  HeadContent,
  Scripts,
} from '@tanstack/react-router'

export const Route = createRootRoute({
  head: () => ({
    meta: [
      { charSet: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { title: '<project-name>' },
    ],
    links: [{ rel: 'icon', href: '/favicon.ico' }],
  }),
  component: RootComponent,
  notFoundComponent: NotFoundComponent,
})

function RootComponent() {
  return (
    <RootDocument>
      <Outlet />
    </RootDocument>
  )
}

function NotFoundComponent() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold">404</h1>
        <p className="mt-2 text-muted-foreground">Page not found</p>
      </div>
    </div>
  )
}

function RootDocument({ children }: Readonly<{ children: ReactNode }>) {
  return (
    <html lang="en">
      <head>
        <HeadContent />
      </head>
      <body>
        {children}
        <Scripts />
      </body>
    </html>
  )
}
```

### `src/routes/index.tsx`
```tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/')({
  component: Home,
})

function Home() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold tracking-tight"><project-name></h1>
        <p className="mt-2 text-muted-foreground">Ready to build.</p>
      </div>
    </div>
  )
}
```

---

## Step 9 — Generate `routeTree.gen.ts`

Run the dev server briefly — it will auto-generate `src/routeTree.gen.ts` on first start, then stop it.

```bash
# Start dev server, wait ~5 seconds, then Ctrl+C
bun run dev
```

Confirm `src/routeTree.gen.ts` now exists before proceeding.

---

## Step 10 — Init shadcn/ui

```bash
bunx --bun shadcn@latest init --preset aLrO8A --base base --template start
```

This preset configures:
- `base-nova` style with OKLCH colour system (light + dark)
- Hugeicons as the icon library
- Inter Variable font
- Full CSS variable theming in `src/styles/globals.css`
- `@/*` import aliases in `components.json`

---

## Step 11 — Install Hugeicons

```bash
bun add @hugeicons/react @hugeicons/core-free-icons
```

Usage pattern going forward:
```tsx
import { HugeiconsIcon } from '@hugeicons/react'
import { HomeIcon } from '@hugeicons/core-free-icons'

<HugeiconsIcon icon={HomeIcon} />
```

---

## Step 12 — Typecheck

```bash
bun run typecheck
```

Must exit with **zero errors**. If there are errors, fix them before declaring the project ready.

---

## Done — Summary

Tell the user:

1. The full directory structure that was created
2. How to start the dev server: `bun run dev` → `http://localhost:5173`
3. Key next steps:
   - Add shadcn components: `bunx --bun shadcn@latest add <component>`
   - Add Cloudflare bindings in `wrangler.jsonc`, then regenerate types with `bun run cf-typegen`
   - Write server functions with `createServerFn` from `@tanstack/react-start`
   - Deploy: `bun run deploy`

---

## Gotchas to avoid

- **Never use Node.js APIs** — no `fs`, `path`, `process.env`, etc. This is Cloudflare Workers only.
- **Never write raw CSS** — Tailwind classes only; theme vars go in `globals.css` under `@theme inline`.
- **Never use npm/yarn/pnpm** — bun only.
- **Don't edit `routeTree.gen.ts`** — it is auto-generated by the Vite plugin.
- **Vite plugin order** — changing the order of plugins in `vite.config.ts` will break the build.
- **`verbatimModuleSyntax`** — never add this to `tsconfig.json`.
- **`getRouter` is required** — TanStack Start v1.166+ calls `getRouter()` at runtime on the router entry. Without it, every request crashes with `entries.routerEntry.getRouter is not a function`.
