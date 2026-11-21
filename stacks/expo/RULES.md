# Expo SDK 58+ (React Native) Engineering Standards

Complete coding style, configuration, and architectural rules for mobile-only (iOS + Android) Expo SDK 58+ projects with Expo Router. These rules extend [`common/BASE_RULES.md`](../../common/BASE_RULES.md).

---

## 1. Mandatory Freshness & Breaking-Change Check (Pre-Coding Gate)

> **STOP & VERIFY BEFORE SCAFFOLDING OR CODING.**

Before running `create-expo-app`, upgrading Expo SDKs, installing native modules, or writing feature code:

1. **Check installed versions and release notes**: inspect `package.json` (`expo`, `expo-router`, `react-native`, `react`, `typescript`, `eslint-config-expo`) and review the latest Expo SDK changelog (`https://expo.dev/changelog`) and React Native release notes. (Since Expo SDK 55+, all Expo SDK packages share the same major version as `expo`.)
2. **Check CLI & ecosystem drift**: verify `npx --yes create-expo-app@latest --help`, `npx expo-doctor`, Expo Router routing/config conventions, React Native's Strict TypeScript API, and `eslint-config-expo` flat config support.
3. **If anything changed, deprecated, or conflicts/breaks against these rules**:
   - **STOP before coding.**
   - Explain clearly what changed (documented rule vs. current framework/CLI behavior).
   - Provide a concrete recommendation for the active project.
   - Ask the user how to proceed, and offer to update `Jacksonkao97/dev-standards`.

---

## 2. Toolchain & Core Defaults

| Layer | Standard |
| --- | --- |
| Target platforms | Mobile-only (**iOS + Android**) |
| Package management | `npm` for pure JS/dev tooling; **`npx expo install <pkg>`** for Expo SDK & native packages |
| Framework & router | Expo SDK 58+ + React Native 0.88+ + React 19+ + Expo Router (`src/app/`) |
| Bundler | Metro (Expo default) |
| React Compiler | Enabled via `app.json` (`experiments.reactCompiler: true`) |
| Typed routes | Enabled via `app.json` (`experiments.typedRoutes: true`) |
| Language | TypeScript (`strict: true`, `noUncheckedIndexedAccess: true`, extending `expo/tsconfig.base`) |
| Path aliases | `@/*` -> `./src/*`, `@/assets/*` -> `./assets/*` |
| Linter | **Expo official ESLint** (`eslint.config.js` extending `eslint-config-expo/flat` + `eslint-config-prettier/flat`) |
| Formatter | **Prettier** (`.prettierrc`) |
| Validation | `zod` |
| Styling & icons | Built-in React Native `StyleSheet.create` + `expo-symbols` |

### App Configuration (`app.json`) & Continuous Native Generation (CNG)

Keep static **`app.json`** during bootstrap so `npx expo install` can automatically register config plugins. Ensure mobile-only defaults and compiler/router flags are enabled while preserving generated app icons and splash plugins:

```json
{
  "expo": {
    "platforms": ["ios", "android"],
    "userInterfaceStyle": "automatic",
    "experiments": {
      "typedRoutes": true,
      "reactCompiler": true
    }
  }
}
```

**Configuration & native workflow rules:**

- **Dynamic config on demand**: convert `app.json` to `app.config.ts` only when the project requires dynamic environment-based configuration (for example, dev/staging/prod bundle identifiers, display names, or conditional plugins).
- **Continuous Native Generation (CNG)**: keep `/ios` and `/android` gitignored and generated via `npx expo prebuild` / EAS Build. Configure native settings through Expo config plugins rather than editing native folders manually.
- **SDK-aligned package installation**: always install Expo SDK and native React Native libraries with `npx expo install <pkg>` so Expo resolves versions compatible with the installed SDK.
- **Development builds over Expo Go**: Expo Go is acceptable for quick prototyping, but prefer **development builds** (`expo-dev-client`) for production apps and whenever custom native modules are introduced.
- **React Compiler & RN Strict TypeScript API**:
  - With `reactCompiler: true` enabled (and `eslint-plugin-react-hooks@7` enforcing Rules of React), do not hand-write `useMemo`, `useCallback`, or `React.memo` by default; reserve manual memoization for profiled hot spots only.
  - React Native 0.87+ enforces the Strict TypeScript API by default: never deep-import from `react-native/Libraries/*`, and type component refs with modern instance types (`ViewInstance`, `TextInputInstance`, etc.).

---

## 3. Feature-Based Hybrid Folder Architecture & Rule of Two

Organize code under `src/` using a feature-based hybrid structure that keeps `src/app/` strictly dedicated to Expo Router routes:

```text
src/
├── app/                     # Expo Router routes & layouts ONLY (keep screens thin)
│   ├── _layout.tsx          # Root navigator layout (providers, Stack / NativeTabs)
│   └── index.tsx            # Initial screen route
├── features/
│   └── <feature>/
│       ├── components/      # Components used by THIS feature only
│       ├── hooks/           # Hooks used by THIS feature only
│       └── lib/             # Helpers, schemas, API functions for THIS feature only
├── components/              # Shared components across 2+ features
├── hooks/                   # Shared React hooks across 2+ features
├── lib/                     # Shared utilities, env schema, theme tokens, API/storage helpers
└── types/                   # Shared TypeScript types
```

### Expo Router `src/app/` Discipline

- **Every file in `src/app/` is a route**: unlike Next.js App Router (which ignores `_components/` private folders), Expo Router treats every file inside `src/app/` as a route or special router file. Never put components, hooks, utilities, or schemas inside `src/app/`.
- **Keep route files thin**: wire route params, navigation headers/options, and screen composition in `src/app/`, while keeping feature UI and domain logic inside `src/features/<feature>/`.
- **Keep `_layout.tsx` lean**: layouts wrap active navigators; avoid heavy data fetching or expensive work directly inside root layouts.

### The Rule of Two

- **Used by 1 feature only**: keep inside `src/features/<feature>/components/`, `hooks/`, or `lib/`.
- **Used by 2 or more features**: promote to top-level `src/components/`, `src/hooks/`, `src/lib/`, or `src/types/`.
- **No cross-feature private imports**: never import from another feature's `components`, `hooks`, or `lib` directory. If a second feature needs it, promote it first.

---

## 4. Styling, Layout & Native UI

- **Lean default**: built-in React Native **`StyleSheet.create`** with zero extra styling dependencies. Colocate component styles at the bottom of the file (`const styles = StyleSheet.create({ ... })`), and extract shared colors, spacing, and typography tokens into `src/lib/` when reused.
- **Icons**: standardize on **`expo-symbols`** (already included in the default SDK 58 template—native SF Symbols on iOS and Material Symbols via the `md` prop on Android). Add `lucide-react-native` + `react-native-svg` (via `npx expo install react-native-svg && npm install lucide-react-native`) on demand when custom cross-platform vector icons are needed.
- **Native-first primitives**:
  - Use **`expo-image`** over React Native's built-in `Image` for caching, placeholders, and memory efficiency.
  - Use **`SafeAreaView`** and **`useSafeAreaInsets`** from `react-native-safe-area-context` (never React Native's legacy iOS-only `SafeAreaView`) so layouts adapt cleanly across notches, Dynamic Island, and resizable/foldable screens.
  - Avoid React Native's deprecated `ImageBackground` (prefer a `View` with an absolutely positioned `Image` from `expo-image`).
  - For large scrollable lists, add **`@shopify/flash-list`** or **`@expo/ui`** recycling lists on demand rather than unoptimized `ScrollView` loops.
- **Styling & UI kits on demand**: add **Uniwind** (native Tailwind CSS v4 + `prettier-plugin-tailwindcss`), **NativeWind**, **react-native-unistyles**, or **`@expo/ui`** (SwiftUI / Jetpack Compose primitives) per project only when product needs justify the dependency.

---

## 5. Navigation, Data Fetching, State, Storage & Boundaries

### Navigation (Expo Router)

- Rely on **typed routes** (`experiments.typedRoutes: true`) with absolute paths (`<Link href="/orders/123" />` or `router.push({ pathname: "/orders/[id]", params: { id } })`), typed `useLocalSearchParams<...>()` generics, and route groups such as `(tabs)/` and `(auth)/`.
- Validate untrusted dynamic route params and incoming deep-link query params with **`zod`** before using them in API requests or state.

### Remote Data Fetching & Mutations

- **Avoid ad-hoc `useEffect` data fetching**: do not hand-roll loading/error/race-condition state inside `useEffect`.
- **Standard on demand**: standardize on **TanStack Query** (`@tanstack/react-query`) as soon as the app communicates with remote APIs (caching, deduplication, retries, background refetching, mutations, and invalidation).
- **Validate API boundaries with `zod`**: treat every external API response and form payload as untrusted and parse it with `zod` before passing it into UI state.
- **Explicit boundary errors**: return a typed discriminated union or raise typed API error classes instead of swallowing failures silently:

```ts
export type ApiResult<T> =
  | { ok: true; data: T }
  | { ok: false; error: string; status?: number };
```

### Client State Management

- **Local React state & route params first**: model screen-scoped filters or identifiers in Expo Router params and ephemeral UI state in local React state (`useState`, `useReducer`).
- **External client state only when justified**: add **Zustand** only when complex cross-tree client state genuinely demands it.

### Local Storage & Persistence

- **Secrets, tokens & credentials**: always store sensitive values (auth access/refresh tokens, encryption keys, private session identifiers) in **`expo-secure-store`** (backed by iOS Keychain and Android Keystore). Never store secrets in unencrypted storage.
- **Non-sensitive local data (on demand)**: standardize on **`expo-sqlite`** for structured relational data as well as synchronous key-value storage (`expo-sqlite/kv-store`, works in both Expo Go and development builds). Consider **`react-native-mmkv`** when a development-build app needs ultra-high-throughput JSI key-value storage.

### Client-Side Environment Variables & Secret Hygiene

- **Zero secrets in `EXPO_PUBLIC_*`**: every `EXPO_PUBLIC_*` variable is statically inlined into the client mobile bundle and can be extracted from the IPA/APK binary. **Never** put API secrets, private keys, signing secrets, or database credentials in `EXPO_PUBLIC_*` variables or app code.
- **Static reads + centralized `zod` validation**: Metro only inlines static `process.env.EXPO_PUBLIC_*` property accesses (and `eslint-plugin-expo` enforces `expo/no-env-var-destructuring` and `expo/no-dynamic-env-var`). Validate public runtime config once in `src/lib/env.ts` by mapping each variable explicitly:

```ts
import { z } from "zod";

const envSchema = z.object({
  EXPO_PUBLIC_API_BASE_URL: z.url(),
});

export const env = envSchema.parse({
  EXPO_PUBLIC_API_BASE_URL: process.env.EXPO_PUBLIC_API_BASE_URL,
});
```

---

## 6. Backend API, Authentication & Distribution (EAS)

- **Intentionally NOT locked in global standards.**
- Choose the backend API architecture, authentication provider, and EAS Build / EAS Update / EAS Submit workflow per project during `/plan`.
- **Mobile security defaults**:
  - Never treat client-side navigation guards or hidden screens as authorization—always enforce authentication and permissions on the backend API.
  - Use OAuth 2.0 + PKCE (via `expo-auth-session` / `expo-web-browser`) or backend-issued short-lived tokens stored in `expo-secure-store`.
  - Never commit `.env*`, Android keystores (`.jks`, `.keystore`), Apple signing keys/certificates (`.p8`, `.p12`), or provisioning profiles (`.mobileprovision`).

---

## 7. TypeScript & Naming Conventions

### File Naming

- Use **`kebab-case`** for all files and folders (e.g., `order-card.tsx`, `use-debounce.ts`, `format-currency.ts`).
- **Exception**: Expo Router reserved conventions (`_layout.tsx`, `index.tsx`, `+not-found.tsx`, `[param].tsx`, `[...slug].tsx`, `(group)/`, platform suffixes like `*.ios.tsx` / `*.android.tsx`) and root config files (`eslint.config.js`, `app.config.ts`, `metro.config.js`, `babel.config.js`).

### Symbol Naming

| Construct | Convention | Example |
| --- | --- | --- |
| React components & TS types | `PascalCase` | `OrderSummary`, `ApiResult` |
| Functions, hooks, variables | `camelCase` | `formatCurrency`, `useDebouncedValue` |
| Module-level constants | `UPPER_SNAKE_CASE` | `MAX_UPLOAD_BYTES`, `DEFAULT_PAGE_SIZE` |

### Exports, Types & Functions

- **Named exports everywhere**: use named exports for components, hooks, utilities, and types. Only use `export default` where Expo Router requires it (route screens and `_layout.tsx` files inside `src/app/`) or where tool config files require it (`eslint.config.js`, `app.config.ts`).
- **Prefer `type` over `interface`**: use `type` aliases consistently (reserve `interface` only when module augmentation / declaration merging is required).
- **Zero `any`**: never use `any`. Use `unknown` plus narrowing or `zod` parsing.
- **Explicit `import type`**: use `import type { ... }` for type-only imports (enforced by ESLint).
- **`noUncheckedIndexedAccess` discipline**: treat indexed array/record access as `T | undefined` and narrow safely instead of scattering non-null assertions (`!`).
- **Function declarations**: use `function` declarations for top-level React components and helper functions (`export function OrderCard(...) {}`).

---

## 8. Testing Strategy

- **Lean bootstrap**: do not pre-install test runners or boilerplate test suites during project bootstrap.
- **Standard test stack when added**:
  - **`jest-expo` + `@testing-library/react-native`** for unit and component tests.
  - **Maestro** for end-to-end (E2E) mobile UI flows on iOS simulators and Android emulators.
- Once tests exist in a repository, they become part of the Definition of Done and must pass before claiming completion.

---

## 9. Verification & Definition of Done

Before committing or claiming work is done, run the full check script and verify exit code `0`:

```bash
npm run check
# Runs: npm run typecheck && npm run lint && npm run format:check && npm run build
```

After dependency upgrades or `app.json` / native plugin changes, also run:

```bash
npm run doctor
# Runs: npx --yes expo-doctor
```

If tests are configured in the project, run them as well and verify all pass.
