# Project Engineering Rules (React 19 + Vite 8)

Tool-agnostic engineering standards for this repository. Follow every rule below unless the user explicitly approves an exception.

## 1. Mandatory Freshness & Breaking-Change Check (Pre-Coding Gate)

Before scaffolding, upgrading dependencies, or writing feature code:

1. Inspect the installed React, Vite, TypeScript, Tailwind CSS, and Oxlint versions in `package.json`.
2. Check official release notes and docs when adding or upgrading packages (including Vite 8 + `@vitejs/plugin-react` React Compiler support—such as whether Oxc-native `oxc-transform-react` / `react({ compiler: true })` has replaced the Babel preset as the stable default).
3. Check whether any CLI flag, config option, API, or pattern in these rules has changed, been deprecated, or conflicts/breaks with the installed version.
4. If anything changed, deprecated, or conflicts/breaks: **STOP before coding**. Explain clearly what changed, provide a concrete recommendation, and ask the user how to proceed (including offering to update `Jacksonkao97/dev-standards`).

## 2. Stack & Configuration Defaults

- **Package manager & bundler**: `npm` + Vite 8 (Rolldown + Oxc) + React 19 SPA (`@/*` -> `./src/*`).
- **TypeScript**: `strict: true` and `noUncheckedIndexedAccess: true` in `tsconfig.app.json` and `tsconfig.node.json`. Define relative `"paths": { "@/*": ["./src/*"] }` without `baseUrl` (removed in TypeScript 7).
- **Linting & formatting**: Oxlint (`.oxlintrc.json`) + Prettier with `prettier-plugin-tailwindcss` (`.prettierrc` pointing `tailwindStylesheet` at `./src/index.css`).
- **Vite config (`vite.config.ts`)**:
  - Enable `react()`, `babel({ presets: [reactCompilerPreset()] })` for React Compiler auto-memoization, and `tailwindcss()` (`@tailwindcss/vite`).
  - Configure explicit `resolve.alias` mapping `@` -> `fileURLToPath(new URL("./src", import.meta.url))` so aliases resolve identically across Vite dev/build, CSS, and Vitest.
  - Because React Compiler handles memoization automatically, avoid manual `useMemo`, `useCallback`, and `React.memo` unless profiling proves a specific need.

## 3. Commands

```bash
npm run dev           # Start Vite dev server
npm run preview       # Preview the production build locally
npm run typecheck     # Run tsc -b across app and node tsconfigs
npm run lint          # Run Oxlint across the repo
npm run lint:fix      # Run Oxlint with auto-fix
npm run format        # Format files with Prettier
npm run format:check  # Verify Prettier formatting
npm run build         # Run production Vite build
npm run check         # Full verification gate: typecheck, lint, format:check, build
```

## 4. Feature-Based Hybrid Architecture & Rule of Two

```text
src/
├── main.tsx                 # App entry point (mounts <App /> safely)
├── app.tsx                  # Root application shell (named export App)
├── index.css                # Tailwind CSS v4 entry (@import "tailwindcss";)
├── routes/                  # Added on demand if TanStack Router is installed (keep route files thin)
├── features/
│   └── <feature>/
│       ├── components/      # Components used by THIS feature only
│       ├── hooks/           # Hooks used by THIS feature only
│       └── lib/             # Helpers, schemas, API functions for THIS feature only
├── components/              # Shared UI across 2+ features (src/components/ui/ if shadcn/ui is added)
├── hooks/                   # Shared React hooks across 2+ features
├── lib/                     # Shared utilities, env schema, HTTP/API client helpers
└── types/                   # Shared TypeScript types
```

- **Rule of Two**:
  - Used by **1 feature only** -> keep inside `src/features/<feature>/{components,hooks,lib}/`.
  - Used by **2+ features** -> promote to top-level `src/{components,hooks,lib,types}/`.
- **Strict isolation**: never import from another feature's `components`, `hooks`, or `lib` directory. If a second feature needs it, promote it first.
- **TanStack Router caveat**: if file-based `src/routes/` is added later, note that TanStack Router uses a `-` prefix (e.g., `-components/`) to ignore colocated files and reserves `_` for pathless layout routes; also add `src/routeTree.gen.ts` to `.oxlintrc.json` and `.prettierignore`.

## 5. Styling & UI

- **Lean default**: build with **Tailwind CSS v4** (`@tailwindcss/vite` + `@import "tailwindcss";` in `src/index.css`) and **`lucide-react`** icons.
- **Primitives on demand**: install `shadcn/ui` (into `src/components/ui/`) or headless accessible primitives only when the product requires complex interactive widgets (dialogs, comboboxes, popovers, menus, date pickers).
- Keep Tailwind class ordering consistent via `prettier-plugin-tailwindcss`.

## 6. Routing, Data Fetching, State & Environment Variables

- **Client-side routing**: keep bootstrap router-free; add **TanStack Router** (`@tanstack/react-router`) on demand for multi-page SPAs (typed routes + Zod-validated URL search params).
- **Remote server state**: avoid ad-hoc `useEffect` data fetching. Standardize on **TanStack Query** (`@tanstack/react-query`) on demand for caching, mutations, retries, and invalidation.
- **Client state**: prefer URL search params and local React state (`useState`, `useReducer`) first. Add **Zustand** only when complex cross-tree client state genuinely requires it.
- **API boundaries & error handling**: validate untrusted API responses and user input with `zod` at boundaries, and model expected failures explicitly with typed errors or a discriminated union (`{ ok: true; data: T } | { ok: false; error: string }`).
- **Environment variables**: every `VITE_*` variable is bundled into public client JavaScript—**never place secrets, API private keys, or database credentials in `VITE_*`**. Validate `import.meta.env` once in `src/lib/env.ts` with `zod`:

```ts
import { z } from "zod";

const envSchema = z.object({
  VITE_API_BASE_URL: z.url(),
});

export const env = envSchema.parse(import.meta.env);
```

- **Backend API, Auth & Hosting**: intentionally project-specific; decide during planning. Never enforce authorization solely in client UI/routes—always enforce auth on the backend API.

## 7. TypeScript & Naming Conventions

| Construct          | Convention         | Notes                                                                                                         |
| ------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------- |
| Files & folders    | `kebab-case`       | Except tool configs (`vite.config.ts`) and framework-generated files (e.g., `__root.tsx`, `routeTree.gen.ts`) |
| Components & types | `PascalCase`       | e.g., `OrderSummary`, `ApiResult`                                                                             |
| Functions & vars   | `camelCase`        | e.g., `formatCurrency`, `useDebouncedValue`                                                                   |
| Module constants   | `UPPER_SNAKE_CASE` | e.g., `DEFAULT_PAGE_SIZE`, `MAX_UPLOAD_BYTES`                                                                 |

- **Exports**: use named exports everywhere (`export function App()`, components, hooks, utilities, types). Only use `export default` where tool/framework config requires it (`vite.config.ts`).
- **Types**: prefer `type` over `interface`, zero `any` (use `unknown` + narrowing/`zod`), explicit `import type` for type-only imports, and narrow `T | undefined` safely under `noUncheckedIndexedAccess` instead of non-null assertions (`!`).
- **Functions**: use `function` declarations for top-level React components and helper functions.
- **Modularity & error handling**: keep modules small and focused, handle errors explicitly (no empty `catch` blocks), and remove dead code, unused imports, and stray `console.log` statements.

## 8. Testing

- Keep the bootstrap lean: do not pre-install test runners until tests are needed.
- When adding tests, standardize on **Vitest** (unit/integration) and **Playwright** (E2E).
- Once tests exist in the repository, they become part of the Definition of Done.

## 9. Security, Git Workflow & Definition of Done

- **Security**:
  - Never commit secrets or `.env*` files (provide `.env.example` with placeholder keys only if needed) and never expose secrets via `VITE_*` variables.
  - Validate all untrusted external input (forms, URL params, API payloads, env) at boundaries with `zod`.
  - Enforce authentication and authorization on the backend; prefer `HttpOnly` cookie sessions or OAuth/PKCE over storing sensitive tokens in `localStorage`.
- **Git**:
  - Write small, focused commits using Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `test:`).
- **Definition of Done**:
  - Never claim a task is complete until `npm run check` (`typecheck`, `lint`, `format:check`, `build`) and all existing tests pass with exit code `0`.
