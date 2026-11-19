# Next.js 16+ Project Bootstrap Recipe

Reproducible step-by-step recipe to scaffold a fresh Next.js 16+ App Router project using upstream `create-next-app@latest` plus the version-safe overlays in [`files/`](files/).

---

## Step 0: Mandatory Freshness & Breaking-Change Check

Before running any scaffold command:

1. Check the latest `create-next-app` version and flags:
   ```bash
   npx --yes create-next-app@latest --version
   npx --yes create-next-app@latest --help
   ```
2. Check Next.js release notes (and, right after scaffolding, `node_modules/next/dist/docs/`) for any breaking changes to:
   - `create-next-app` CLI flags (e.g., linter flags, bundler defaults, `AGENTS.md` generation)
   - `next.config.ts` options (`reactCompiler`, `typedRoutes`, `cacheComponents`, `partialPrefetching`)
   - Oxlint rules/plugins or `prettier-plugin-tailwindcss` Tailwind v4 integration
3. **If any flag, config option, or dependency changed, was deprecated, or conflicts/breaks**: **STOP**, explain what changed, recommend an update, and ask the user how to proceed (offering to update `Jacksonkao97/dev-standards`).

> Verified against `next@16.4.6` / `create-next-app@16.4.6` (where Turbopack is the default bundler, `--no-eslint` is the canonical flag to skip ESLint/Biome, and `--no-agents-md` skips the default stub `AGENTS.md`).

---

## Step 1: Scaffold with `create-next-app@latest`

Run from the parent directory where you want `<project-name>` created:

```bash
npx create-next-app@latest <project-name> \
  --ts \
  --tailwind \
  --no-eslint \
  --app \
  --src-dir \
  --react-compiler \
  --cache-components \
  --import-alias "@/*" \
  --empty \
  --use-npm \
  --no-agents-md \
  --no-agent-feedback
```

Notes:

- Turbopack is the default bundler in Next.js 16+ (no `--turbopack` flag needed; `--rspack` is only for opting out).
- `--no-eslint` skips installing ESLint/Biome cleanly (older `create-next-app` versions also accepted `--no-linter`).
- `--no-agents-md` prevents creating a stub `AGENTS.md` right before Step 3 copies our full `AGENTS.md`. Note that running `next dev` under an AI coding agent may later append Next.js's managed `<!-- BEGIN:nextjs-agent-rules -->` block to `AGENTS.md`; keep that block committed so `git status` stays clean.

Enter the newly created directory (`<project-name>`) for Steps 2–5.

---

## Step 2: Install Core & Dev Dependencies

```bash
npm install lucide-react zod server-only
npm install -D oxlint prettier prettier-plugin-tailwindcss
```

Do **not** pre-install database clients, auth providers, UI component kits (`shadcn/ui`), or test runners (`vitest`, `playwright`) during bootstrap—add those per project on demand.

---

## Step 3: Copy Version-Safe Overlay Files

Copy every file from `stacks/nextjs/files/` into the new project root:

- `AGENTS.md`
- `CLAUDE.md`
- `.oxlintrc.json`
- `.prettierrc`
- `.prettierignore`
- `.editorconfig`

For example, if `dev-standards` is checked out locally at `<path-to-dev-standards>`:

```bash
cp -a <path-to-dev-standards>/stacks/nextjs/files/. ./
```

(Or fetch each file directly from `https://github.com/Jacksonkao97/dev-standards/tree/main/stacks/nextjs/files`.)

---

## Step 4: Apply Project Config & Folder Structure

1. **Configure `next.config.ts`**:
   Ensure these four top-level flags are enabled while **preserving** any generated bundler/loader rules (such as `turbopack.rules` for `@tailwindcss/turbopack`):

   ```ts
   import type { NextConfig } from "next";

   const nextConfig: NextConfig = {
     cacheComponents: true,
     partialPrefetching: true,
     reactCompiler: true,
     typedRoutes: true,
     // Preserve generated turbopack rules here (e.g., @tailwindcss/turbopack)
   };

   export default nextConfig;
   ```

   > If the project targets static hosting (`output: "export"`), omit/disable `cacheComponents` and `partialPrefetching`. While `cacheComponents: true` is active, never export a `runtime` route segment config.

2. **Strengthen `tsconfig.json`**:
   Add `"noUncheckedIndexedAccess": true` inside `compilerOptions` (keep `strict: true`, `@/*` -> `./src/*`, and all generated Next.js entries).

3. **Update `package.json` scripts**:
   Add the standard verification and formatting scripts alongside `dev`, `build`, and `start`:

   ```json
   {
     "scripts": {
       "dev": "next dev",
       "build": "next build",
       "start": "next start",
       "typecheck": "next typegen && tsc --noEmit",
       "lint": "oxlint",
       "lint:fix": "oxlint --fix",
       "format": "prettier --write .",
       "format:check": "prettier --check .",
       "check": "npm run typecheck && npm run lint && npm run format:check && npm run build"
     }
   }
   ```

4. **Create shared top-level directories**:

   ```bash
   mkdir -p src/components src/hooks src/lib src/types
   ```

   (Leave empty directories untracked until needed, or add `.gitkeep` only if you want empty folders committed upfront.)

5. **Normalize formatting across scaffolded files**:

   ```bash
   npm run format
   ```

---

## Step 5: Verify End-to-End & Commit

Run the full verification gate and confirm exit code `0` with zero errors and zero lint warnings:

```bash
npm run check
```

(`npm run check` runs `npm run typecheck && npm run lint && npm run format:check && npm run build`.)

Once `npm run check` passes cleanly, inspect `git status` and create the bootstrap commit:

```bash
git add -A
git commit -m "chore: bootstrap nextjs project"
```
