# Project Engineering Rules (Expo SDK 58+ / React Native)

Tool-agnostic engineering standards for this repository. Follow every rule below unless the user explicitly approves an exception.

## 1. Mandatory Freshness & Breaking-Change Check (Pre-Coding Gate)

Before scaffolding, upgrading dependencies, or writing feature code:

1. Inspect the installed `expo`, `expo-router`, `react-native`, `react`, `typescript`, and `eslint-config-expo` versions in `package.json`. (Since Expo SDK 55+, all Expo SDK packages share the same major version as `expo`.)
2. Verify current Expo SDK release notes (`expo.dev/changelog`), official Expo docs, and `npx expo-doctor` when adding native packages or upgrading SDKs.
3. Check whether any CLI flag, config option, API, or pattern in these rules has changed, been deprecated, or conflicts/breaks with the installed SDK.
4. If anything changed, deprecated, or conflicts/breaks: **STOP before coding**. Explain clearly what changed, provide a concrete recommendation, and ask the user how to proceed (including offering to update `Jacksonkao97/dev-standards`).

## 2. Stack & Configuration Defaults

- **Target platforms**: mobile-only (**iOS + Android**). Do not add web targets, `*.web.tsx` variants, or web-only dependencies unless the project explicitly adopts web later.
- **Package management**: use `npm` for pure JS/dev tooling and **`npx expo install <pkg>`** for all Expo SDK and native React Native dependencies so SDK-compatible versions are selected.
- **Framework & bundler**: Expo SDK 58+ + React Native 0.88+ + React 19+ + Expo Router (`src/app/`) + Metro (`@/*` -> `./src/*`, `@/assets/*` -> `./assets/*`).
- **App config & Continuous Native Generation (CNG)**:
  - Keep static **`app.json`** by default (`platforms: ["ios", "android"]`, `userInterfaceStyle: "automatic"`, `experiments.typedRoutes: true`, `experiments.reactCompiler: true`) so `npx expo install` can auto-configure plugins. Convert to `app.config.ts` only when dynamic env/flavor config is needed.
  - Treat `/ios` and `/android` as generated artifacts (gitignored; regenerated via `npx expo prebuild` / EAS Build)—never hand-edit native folders in a CNG project.
  - Prefer **development builds** (`expo-dev-client`) over Expo Go for production apps and any custom native modules.
- **TypeScript**: `strict: true` and `noUncheckedIndexedAccess: true` in `tsconfig.json` (extending `expo/tsconfig.base`). Follow React Native's Strict TypeScript API (no deep `react-native/Libraries/*` imports; use modern ref types like `ViewInstance` / `TextInputInstance`).
- **React Compiler**: enabled via `experiments.reactCompiler: true`. Avoid manual `useMemo`, `useCallback`, and `React.memo` by default unless profiling proves a specific need.
- **Linting & formatting**: Expo official ESLint (`eslint.config.js` extending `eslint-config-expo/flat` + `eslint-config-prettier/flat`) and Prettier (`.prettierrc`).

## 3. Commands

```bash
npm start             # Start Expo Metro dev server
npm run ios           # Launch on iOS simulator / device
npm run android       # Launch on Android emulator / device
npm run typecheck     # Generate Expo Router types and run tsc --noEmit
npm run lint          # Run Expo ESLint (expo lint)
npm run lint:fix      # Run Expo ESLint with auto-fix
npm run format        # Format files with Prettier
npm run format:check  # Verify Prettier formatting
npm run build         # Bundle iOS + Android JS/assets via expo export
npm run doctor        # Run Expo SDK & config health check (npx --yes expo-doctor)
npm run check         # Full verification gate: typecheck, lint, format:check, build
```

## 4. Feature-Based Hybrid Architecture & Rule of Two

```text
src/
├── app/                     # Expo Router routes & layouts ONLY (keep screens thin)
│   ├── _layout.tsx          # Root navigator layout (providers, Stack/Tabs)
│   └── index.tsx            # Initial screen route
├── features/
│   └── <feature>/
│       ├── components/      # Components used by THIS feature only
│       ├── hooks/           # Hooks used by THIS feature only
│       └── lib/             # Helpers, schemas, API functions for THIS feature only
├── components/              # Shared UI across 2+ features
├── hooks/                   # Shared React hooks across 2+ features
├── lib/                     # Shared utilities, env schema, HTTP/storage helpers
└── types/                   # Shared TypeScript types
```

- **Routes-only `src/app/`**: Expo Router treats **every** file in `src/app/` as a route. Never place components, hooks, schemas, or helpers inside `src/app/`—keep route files thin and put feature UI/logic under `src/features/<feature>/`.
- **Rule of Two**:
  - Used by **1 feature only** -> keep inside `src/features/<feature>/{components,hooks,lib}/`.
  - Used by **2+ features** -> promote to top-level `src/{components,hooks,lib,types}/`.
- **Strict isolation**: never import from another feature's `components`, `hooks`, or `lib` directory. If a second feature needs it, promote it first.

## 5. Styling, Layout & Native UI

- **Lean default**: built-in React Native **`StyleSheet.create`** (colocate `const styles = StyleSheet.create(...)` at the bottom of the component; place shared theme tokens in `src/lib/`).
- **Icons**: use **`expo-symbols`** (SF Symbols on iOS + Material Symbols via `md` prop on Android) by default. Add `lucide-react-native` + `react-native-svg` on demand when custom cross-platform vector icons are needed.
- **Native-first primitives**: use `expo-image` instead of RN `Image`, `SafeAreaView` / `useSafeAreaInsets` from `react-native-safe-area-context` for safe areas, and `@shopify/flash-list` (or `@expo/ui` recycling lists) on demand for large scrollable lists.
- **Styling & UI kits on demand**: do not pre-install styling frameworks during bootstrap. Add **Uniwind** (Tailwind CSS v4 + `prettier-plugin-tailwindcss`), **NativeWind**, **react-native-unistyles**, or native **`@expo/ui`** (SwiftUI / Jetpack Compose) only when the project calls for it.

## 6. Navigation, Data Fetching, State, Storage & Environment Variables

- **Navigation**: use Expo Router typed routes (`Href`, `useLocalSearchParams<...>()`), route groups (`(tabs)/`, `(auth)/`), and lean `_layout.tsx` files (do not put heavy data fetching in layouts).
- **Remote server state**: avoid ad-hoc `useEffect` data fetching. Standardize on **TanStack Query** (`@tanstack/react-query`) on demand for caching, mutations, retries, and invalidation.
- **Client state**: prefer local React state (`useState`, `useReducer`) and Expo Router route/search params first. Add **Zustand** only when shared cross-tree client state genuinely requires it.
- **API boundaries & error handling**: validate all untrusted API responses, form payloads, and route/deep-link params with `zod` at boundaries, and return explicit typed errors or discriminated unions (`{ ok: true; data: T } | { ok: false; error: string }`).
- **Local storage**:
  - **Secrets & tokens**: always use **`expo-secure-store`** (iOS Keychain / Android Keystore)—never unencrypted storage.
  - **Non-sensitive persistence (on demand)**: standardize on **`expo-sqlite`** (relational SQLite or sync `expo-sqlite/kv-store`), with `react-native-mmkv` as an option for dev-build high-throughput KV needs.
- **Environment variables**: every `EXPO_PUBLIC_*` variable is statically inlined into the client app bundle—**never put API secrets, private keys, or database credentials in `EXPO_PUBLIC_*`**. Because Metro inlines static property reads (and `eslint-plugin-expo` forbids destructuring/dynamic access on `process.env`), map each key explicitly into `zod` in `src/lib/env.ts`:

```ts
import { z } from "zod";

const envSchema = z.object({
  EXPO_PUBLIC_API_BASE_URL: z.url(),
});

export const env = envSchema.parse({
  EXPO_PUBLIC_API_BASE_URL: process.env.EXPO_PUBLIC_API_BASE_URL,
});
```

- **Backend API, Auth & EAS**: intentionally project-specific; decide during planning. Always enforce authentication and authorization on the backend—never rely on client route guards alone.

## 7. TypeScript & Naming Conventions

| Construct          | Convention         | Notes                                                                                                                                     |
| ------------------ | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Files & folders    | `kebab-case`       | Except Expo Router files (`_layout.tsx`, `index.tsx`, `+not-found.tsx`, `[param].tsx`, `[...slug].tsx`, `(group)/`) and root config files |
| Components & types | `PascalCase`       | e.g., `OrderSummary`, `ApiResult`                                                                                                         |
| Functions & vars   | `camelCase`        | e.g., `formatCurrency`, `useDebouncedValue`                                                                                               |
| Module constants   | `UPPER_SNAKE_CASE` | e.g., `DEFAULT_PAGE_SIZE`, `MAX_UPLOAD_BYTES`                                                                                             |

- **Exports**: use named exports everywhere except Expo Router route/layout files (`src/app/**`) and config files that require default exports (`eslint.config.js`, `app.config.ts`).
- **Types**: prefer `type` over `interface`, zero `any` (use `unknown` + narrowing/`zod`), explicit `import type` for type-only imports, and narrow `T | undefined` safely under `noUncheckedIndexedAccess` instead of non-null assertions (`!`).
- **Functions**: use `function` declarations for top-level React components and helper functions.
- **Modularity & error handling**: keep modules small and focused, handle errors explicitly (no empty `catch` blocks), and remove dead code, unused imports, and stray `console.log` statements.

## 8. Testing

- Keep the bootstrap lean: do not pre-install test runners until tests are needed.
- When adding tests, standardize on **`jest-expo` + `@testing-library/react-native`** (unit/component) and **Maestro** (mobile E2E).
- Once tests exist in the repository, they become part of the Definition of Done.

## 9. Security, Git Workflow & Definition of Done

- **Security**:
  - Never commit secrets, `.env*` files, keystores (`.jks`, `.keystore`), signing certificates (`.p8`, `.p12`), or provisioning profiles (`.mobileprovision`).
  - Validate all untrusted external input (forms, deep-link/route params, API payloads, env) at boundaries with `zod`.
  - Store auth tokens in `expo-secure-store`, keep secrets out of `EXPO_PUBLIC_*`, and enforce authorization on the backend.
- **Git**:
  - Write small, focused commits using Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `test:`).
- **Definition of Done**:
  - Never claim a task is complete until `npm run check` (`typecheck`, `lint`, `format:check`, `build`) and all existing tests pass with exit code `0` (and `npm run doctor` passes after dependency or Expo config changes).
