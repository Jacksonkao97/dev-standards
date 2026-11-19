# Project Engineering Rules (Next.js 16+)

Tool-agnostic engineering standards for this repository. Follow every rule below unless the user explicitly approves an exception.

## 1. Mandatory Freshness & Breaking-Change Check (Pre-Coding Gate)

Before scaffolding, upgrading dependencies, or writing feature code:

1. Inspect the installed Next.js and dependency versions in `package.json`.
2. Read the relevant bundled guides in `node_modules/next/dist/docs/` before relying on Next.js APIs, routing conventions, caching directives, or config options; verify latest official docs and release notes when adding or upgrading packages.
3. Check whether any CLI flag, config option, API, or pattern in these rules has changed, been deprecated, or conflicts/breaks with the installed version.
4. If anything changed, deprecated, or conflicts/breaks: **STOP before coding**. Explain clearly what changed, provide a concrete recommendation, and ask the user how to proceed (including offering to update `Jacksonkao97/dev-standards`).

## 2. Stack & Configuration Defaults

- **Package manager & bundler**: `npm` + Next.js 16+ App Router + Turbopack (`@/*` -> `./src/*`).
- **TypeScript**: `strict: true` and `noUncheckedIndexedAccess: true` in `tsconfig.json`.
- **Linting & formatting**: Oxlint (`.oxlintrc.json`) + Prettier with `prettier-plugin-tailwindcss` (`.prettierrc`).
- **Next.js config (`next.config.ts`)**:
  - Enable `reactCompiler: true`, `typedRoutes: true`, `cacheComponents: true`, and `partialPrefetching: true`.
  - Preserve any generated `turbopack` loader rules (such as `@tailwindcss/turbopack`).
  - **Static export caveat**: disable `cacheComponents` and `partialPrefetching` if the project uses `output: "export"`.
  - **Runtime caveat**: never export a `runtime` route segment config (including `runtime = "nodejs"` or `runtime = "edge"`) while `cacheComponents: true` is active.
  - **Caching model**: under Cache Components, data access is dynamic by default. Wrap request-time subtrees in `<Suspense>` boundaries and opt in to caching explicitly with `"use cache"`, `cacheLife`, and `cacheTag`.

## 3. Commands

```bash
npm run dev           # Start local dev server
npm run typecheck     # Run next typegen && tsc --noEmit
npm run lint          # Run Oxlint across the repo
npm run lint:fix      # Run Oxlint with auto-fix
npm run format        # Format files with Prettier
npm run format:check  # Verify Prettier formatting
npm run build         # Run production Next.js build
npm run check         # Full verification gate: typecheck, lint, format:check, build
```

## 4. Hybrid Folder Architecture & Rule of Two

```text
src/
├── app/
│   ├── api/               # Route Handlers only (webhooks, callbacks, external APIs)
│   └── <route>/
│       ├── page.tsx
│       ├── _components/   # Route-private UI components (1 route only)
│       ├── _hooks/        # Route-private client hooks (1 route only)
│       └── _lib/          # Route-private helpers, schemas, Server Actions (1 route only)
├── components/            # Shared UI across 2+ routes (src/components/ui/ if shadcn/ui is added)
├── hooks/                 # Shared React hooks across 2+ routes
├── lib/                   # Shared utilities, env validation, server helpers
└── types/                 # Shared TypeScript types
```

- **Rule of Two**:
  - Used by **1 route only** -> keep inside `src/app/<route>/_components/`, `_hooks/`, or `_lib/`.
  - Used by **2+ routes** -> promote to top-level `src/components/`, `src/hooks/`, `src/lib/`, or `src/types/`.
- **Strict isolation**: never import from another route's `_components`, `_hooks`, or `_lib` directory.

## 5. Styling & UI

- **Lean default**: build with **Tailwind CSS v4** (`@import "tailwindcss";` in `src/app/globals.css`) and **`lucide-react`** icons.
- **Primitives on demand**: install `shadcn/ui` (into `src/components/ui/`) or headless accessible primitives only when the product requires complex interactive widgets (dialogs, comboboxes, popovers, menus, date pickers).
- Keep Tailwind class ordering consistent via `prettier-plugin-tailwindcss`.

## 6. Data Fetching, Mutations & State

- **Server-first**: Server Components are the default. Push `"use client"` down to the smallest interactive leaf component.
- **Server Actions**: validate every input with `zod`, verify auth/permissions on the server, and return a typed discriminated union instead of throwing expected errors across the client boundary:

```ts
export type ActionResult<T> =
  { ok: true; data: T } | { ok: false; error: string; fieldErrors?: Record<string, string[]> };
```

- **Route Handlers (`src/app/api/`)**: reserve for webhooks, OAuth callbacks, health checks, and external HTTP APIs. Use Server Components and Server Actions for internal reads and mutations.
- **Client state**: prefer URL search params (`searchParams`) and local React state first. Introduce Zustand or TanStack Query only when shared cross-tree client state or client cache sync genuinely requires it.
- **Server isolation**: add `import "server-only";` at the top of server modules touching secrets, database clients, or private SDKs.
- **Environment variables**: validate env vars with `zod` in a dedicated `src/lib/env.ts` module rather than scattering raw `process.env` reads across the codebase:

```ts
import "server-only";
import { z } from "zod";

const serverEnvSchema = z.object({
  NODE_ENV: z.enum(["development", "test", "production"]),
});

export const serverEnv = serverEnvSchema.parse(process.env);
```

- **Database & Auth**: intentionally project-specific; follow the architecture decided during planning rather than assuming a fixed provider.

## 7. TypeScript & Naming Conventions

| Construct          | Convention         | Notes                                                                                                                                                                              |
| ------------------ | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Files & folders    | `kebab-case`       | Except Next.js reserved files (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `global-error.tsx`, `not-found.tsx`, `route.ts`, `default.tsx`, `proxy.ts`, `next.config.ts`) |
| Components & types | `PascalCase`       | e.g., `OrderSummary`, `ActionResult`                                                                                                                                               |
| Functions & vars   | `camelCase`        | e.g., `formatCurrency`, `useDebouncedValue`                                                                                                                                        |
| Module constants   | `UPPER_SNAKE_CASE` | e.g., `DEFAULT_PAGE_SIZE`, `MAX_UPLOAD_BYTES`                                                                                                                                      |

- **Exports**: use named exports everywhere except Next.js route, metadata, and config files that require default exports.
- **Types**: prefer `type` over `interface`, zero `any` (use `unknown` + narrowing/`zod`), explicit `import type` for type-only imports, and handle `T | undefined` safely under `noUncheckedIndexedAccess`.
- **Functions**: use `function` declarations for top-level React components and helper functions.
- **Modularity & error handling**: keep modules small and focused, handle errors explicitly (no empty `catch` blocks), and remove dead code, unused imports, and stray `console.log` statements.

## 8. Testing

- Keep the bootstrap lean: do not pre-install test runners until tests are needed.
- When adding tests, standardize on **Vitest** (unit/integration) and **Playwright** (E2E).
- Once tests exist in the repository, they become part of the Definition of Done.

## 9. Security, Git Workflow & Definition of Done

- **Security**:
  - Never commit secrets or `.env*` files; provide `.env.example` with placeholder keys only if needed.
  - Validate all untrusted external input (forms, params, webhooks, headers, env) at boundaries with `zod`.
  - Keep server secrets out of client bundles and enforce auth/authorization on the server.
- **Git**:
  - Write small, focused commits using Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `test:`).
- **Definition of Done**:
  - Never claim a task is complete until `npm run check` (`typecheck`, `lint`, `format:check`, `build`) and all existing tests pass with exit code `0`.
