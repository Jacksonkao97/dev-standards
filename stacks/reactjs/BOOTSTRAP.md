# React 19 + Vite 8 Project Bootstrap Recipe

Reproducible step-by-step recipe to scaffold a fresh client-side React 19 + Vite 8 SPA using upstream `create-vite@latest` (`react-compiler-ts`) plus the version-safe overlays in [`files/`](files/).

---

## Step 0: Mandatory Freshness & Breaking-Change Check

Before running any scaffold command:

1. Check the latest `create-vite` version, flags, and available templates:
   ```bash
   npx --yes create-vite@latest --help
   npm view create-vite version
   npm view vite version
   npm view @vitejs/plugin-react version
   npm view @tailwindcss/vite version
   npm view oxlint version
   ```
2. Check release notes and upstream docs for any breaking changes to:
   - `create-vite` CLI flags or template names (`react-compiler-ts`, `--no-eslint`, `--no-immediate`, `--no-interactive`)
   - `@vitejs/plugin-react` React Compiler integration (including whether Oxc-native `oxc-transform-react` + `react({ compiler: true })` has graduated from experimental to replace `@rolldown/plugin-babel` + `reactCompilerPreset()`)
   - TypeScript 7+ compiler defaults (`baseUrl` removal, `strict`, project references)
   - `@tailwindcss/vite`, `prettier-plugin-tailwindcss`, and Oxlint config/rules
3. **If any flag, config option, or dependency changed, was deprecated, or conflicts/breaks**: **STOP**, explain what changed, recommend an update, and ask the user how to proceed (offering to update `Jacksonkao97/dev-standards`).

> Verified against `create-vite@9.3.4`, `vite@8.3.8`, `react@19.3.1`, `@vitejs/plugin-react@6.3.0`, `@tailwindcss/vite@4.3.5`, `typescript@7.0.x`, and `oxlint@1.94.0` (where Oxlint is the default linter for React templates via `--no-eslint` and `react-compiler-ts` configures `@rolldown/plugin-babel` + `reactCompilerPreset()`).

---

## Step 1: Scaffold with `create-vite@latest`

Run from the parent directory where you want `<project-name>` created:

```bash
npx --yes create-vite@latest <project-name> \
  --template react-compiler-ts \
  --no-eslint \
  --no-immediate \
  --no-interactive
```

Enter the newly created directory (`<project-name>`) and install the scaffolded dependencies for Steps 2–5:

```bash
npm install
```

Notes:

- `--template react-compiler-ts` scaffolds Vite 8 + React 19 + TypeScript with React Compiler pre-wired (`@rolldown/plugin-babel`, `babel-plugin-react-compiler`, and `reactCompilerPreset()`).
- `--no-eslint` keeps `create-vite`'s default Oxlint setup cleanly without installing ESLint.
- `--no-immediate --no-interactive` ensures deterministic, non-interactive scaffolding without starting the dev server immediately.

---

## Step 2: Install Core & Dev Dependencies

```bash
npm install lucide-react zod
npm install -D tailwindcss @tailwindcss/vite prettier prettier-plugin-tailwindcss
```

Notes:

- `create-vite@9`'s React template already includes `oxlint` in `devDependencies`. Confirm `oxlint` is present in `package.json` (if a future template omits it, run `npm install -D oxlint`).
- Do **not** pre-install routers (`@tanstack/react-router`), data-fetching/state libraries (`@tanstack/react-query`, `zustand`), UI component kits (`shadcn/ui`), or test runners (`vitest`, `playwright`) during bootstrap—add those per project on demand.

---

## Step 3: Copy Version-Safe Overlay Files

Copy every file from `stacks/reactjs/files/` into the new project root (this replaces `create-vite`'s minimal `.oxlintrc.json` with the full project lint rules and adds `AGENTS.md`, `CLAUDE.md`, Prettier, and EditorConfig files):

- `AGENTS.md`
- `CLAUDE.md`
- `.oxlintrc.json`
- `.prettierrc`
- `.prettierignore`
- `.editorconfig`

For example, if `dev-standards` is checked out locally at `<path-to-dev-standards>`:

```bash
cp -a <path-to-dev-standards>/stacks/reactjs/files/. ./
```

(Or fetch each file directly from `https://github.com/Jacksonkao97/dev-standards/tree/main/stacks/reactjs/files`.)

---

## Step 4: Apply Project Config, Clean Boilerplate & Folder Structure

1. **Configure `vite.config.ts`**:
   Preserve the generated React + React Compiler plugins, add `tailwindcss()`, and configure the explicit `@` -> `./src` alias:

   ```ts
   import { fileURLToPath, URL } from "node:url";
   import babel from "@rolldown/plugin-babel";
   import tailwindcss from "@tailwindcss/vite";
   import react, { reactCompilerPreset } from "@vitejs/plugin-react";
   import { defineConfig } from "vite";

   export default defineConfig({
     plugins: [react(), babel({ presets: [reactCompilerPreset()] }), tailwindcss()],
     resolve: {
       alias: {
         "@": fileURLToPath(new URL("./src", import.meta.url)),
       },
     },
   });
   ```

2. **Strengthen `tsconfig.app.json` and `tsconfig.node.json`**:
   - In `tsconfig.app.json` inside `compilerOptions`, add:
     ```json
     "strict": true,
     "noUncheckedIndexedAccess": true,
     "paths": {
       "@/*": ["./src/*"]
     }
     ```
     (Do **not** add `"baseUrl"`—TypeScript 7 removed `baseUrl` and treats it as an error.)
   - In `tsconfig.node.json` inside `compilerOptions`, add `"strict": true` and `"noUncheckedIndexedAccess": true`.
   - Preserve all other generated entries (`verbatimModuleSyntax`, `erasableSyntaxOnly`, `types`, `tsBuildInfoFile`, etc.).

3. **Clean starter boilerplate & wire Tailwind CSS v4**:
   - Remove Vite's starter demo files:
     ```bash
     rm -rf src/App.css src/App.tsx src/assets
     ```
   - Replace `src/index.css` with Tailwind CSS v4:
     ```css
     @import "tailwindcss";
     ```
   - Create a minimal `src/app.tsx` using strict `kebab-case` and a named export:
     ```tsx
     export function App() {
       return (
         <main className="flex min-h-screen items-center justify-center p-6">
           <h1 className="text-2xl font-semibold tracking-tight">Ready</h1>
         </main>
       );
     }
     ```
   - Update `src/main.tsx` to import `App` via the `@/*` path alias and narrow `#root` safely without `!`:
     ```tsx
     import { StrictMode } from "react";
     import { createRoot } from "react-dom/client";
     import { App } from "@/app";
     import "./index.css";

     const rootElement = document.getElementById("root");
     if (!rootElement) {
       throw new Error("Root element #root not found");
     }

     createRoot(rootElement).render(
       <StrictMode>
         <App />
       </StrictMode>,
     );
     ```
   - Update `<title>` in `index.html` for the new project (and replace `public/favicon.svg` if desired).

4. **Update `package.json` scripts**:
   Split `typecheck` from `build` and add the standard verification and formatting scripts:

   ```json
   {
     "scripts": {
       "dev": "vite",
       "build": "vite build",
       "preview": "vite preview",
       "typecheck": "tsc -b",
       "lint": "oxlint",
       "lint:fix": "oxlint --fix",
       "format": "prettier --write .",
       "format:check": "prettier --check .",
       "check": "npm run typecheck && npm run lint && npm run format:check && npm run build"
     }
   }
   ```

5. **Create shared top-level directories**:

   ```bash
   mkdir -p src/components src/features src/hooks src/lib src/types
   ```

   (Leave empty directories untracked until needed, or add `.gitkeep` only if you want empty folders committed upfront.)

6. **Normalize formatting across scaffolded files**:

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

Once `npm run check` passes cleanly, initialize git if not already a repository (`git init`), inspect `git status`, and create the bootstrap commit:

```bash
git add -A
git commit -m "chore: bootstrap reactjs project"
```
