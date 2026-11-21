# Expo SDK 58+ Project Bootstrap Recipe

Reproducible step-by-step recipe to scaffold a fresh mobile-only (iOS + Android) Expo SDK 58+ project with Expo Router using upstream `create-expo-app@latest` plus the version-safe overlays in [`files/`](files/).

---

## Step 0: Mandatory Freshness & Breaking-Change Check

Before running any scaffold command:

1. Check the latest `create-expo-app` flags and core package versions:
   ```bash
   npx --yes create-expo-app@latest --help
   npm view create-expo-app version
   npm view expo version
   npm view expo-router version
   npm view react-native version
   npm view eslint-config-expo version
   ```
2. Check Expo SDK release notes (`https://expo.dev/changelog`) and upstream docs for any breaking changes to:
   - `create-expo-app` CLI flags or templates (`--template default`, `--no-agents-md`)
   - `app.json` defaults (`platforms`, `experiments.typedRoutes`, `experiments.reactCompiler`)
   - React Native Strict TypeScript API or `expo/tsconfig.base` defaults
   - `eslint-config-expo/flat` and `eslint-config-prettier/flat` flat config exports
3. **If any flag, config option, or dependency changed, was deprecated, or conflicts/breaks**: **STOP**, explain what changed, recommend an update, and ask the user how to proceed (offering to update `Jacksonkao97/dev-standards`).

> Verified against `create-expo-app@5.0.0`, `expo-template-default@58.0.35`, `expo@58.0.26`, `expo-router@58.0.32`, `react-native@0.88.1`, `react@19.3.0`, `typescript@6.0.x`, `eslint-config-expo@58.0.4`, `eslint-config-prettier@10.1.8`, and `prettier@3.10.1`.

---

## Step 1: Scaffold with `create-expo-app@latest`

Run from the parent directory where you want `<project-name>` created:

```bash
npx --yes create-expo-app@latest <project-name> \
  --template default \
  --no-agents-md
```

Notes:

- `--template default` scaffolds Expo SDK 58+ with TypeScript, Expo Router (`src/app/`), `expo-image`, `expo-symbols`, `react-native-safe-area-context`, `react-native-reanimated`, and `typedRoutes` + `reactCompiler` pre-enabled in `app.json`.
- `--no-agents-md` skips generating Expo's default `AGENTS.md` and `.claude/settings.json` right before Step 3 copies our full `AGENTS.md` and `CLAUDE.md`.

Enter the newly created directory (`<project-name>`) for Steps 2–5.

---

## Step 2: Trim Web-Only Packages & Install Core / Dev Dependencies

Remove web-only dependencies (`react-dom` and `react-native-web` are optional peer dependencies of `expo-router`), install `zod`, and install Expo's official ESLint + Prettier tooling:

```bash
npm uninstall react-dom react-native-web
npm install zod
npx expo install --dev eslint eslint-config-expo
npm install -D prettier eslint-config-prettier
```

Notes:

- Always use `npx expo install <pkg>` when adding Expo SDK or native React Native packages in the future so SDK-compatible versions are selected automatically.
- Do **not** pre-install styling frameworks (`uniwind`, `nativewind`, `react-native-unistyles`), data/state libraries (`@tanstack/react-query`, `zustand`), local storage (`expo-sqlite`, `expo-secure-store`), or test runners (`jest-expo`, `@testing-library/react-native`) during bootstrap—add those per project on demand.

---

## Step 3: Copy Version-Safe Overlay Files

Copy every file from `stacks/expo/files/` into the new project root:

- `AGENTS.md`
- `CLAUDE.md`
- `eslint.config.js`
- `.prettierrc`
- `.prettierignore`
- `.editorconfig`

For example, if `dev-standards` is checked out locally at `<path-to-dev-standards>`:

```bash
cp -a <path-to-dev-standards>/stacks/expo/files/. ./
```

(Or fetch each file directly from `https://github.com/Jacksonkao97/dev-standards/tree/main/stacks/expo/files`.)

---

## Step 4: Apply Mobile-Only Config, Clean Boilerplate & Folder Structure

1. **Configure `app.json` for mobile-only (iOS + Android)**:
   - Add `"platforms": ["ios", "android"]` under `"expo"`.
   - Remove the `"web"` object.
   - Preserve the generated `"name"`, `"slug"`, `"scheme"`, `"icon"`, `"ios"`, `"android"`, `"plugins"` (`expo-router`, `expo-splash-screen`), `"userInterfaceStyle": "automatic"`, and `"experiments": { "typedRoutes": true, "reactCompiler": true }`.

2. **Strengthen `tsconfig.json`**:
   Add `"noUncheckedIndexedAccess": true` inside `compilerOptions` while preserving `"extends": "expo/tsconfig.base"`, `"strict": true`, `"paths"` (`@/*` -> `./src/*`, `@/assets/*` -> `./assets/*`), and `"include"`:

   ```json
   {
     "extends": "expo/tsconfig.base",
     "compilerOptions": {
       "strict": true,
       "noUncheckedIndexedAccess": true,
       "paths": {
         "@/*": ["./src/*"],
         "@/assets/*": ["./assets/*"]
       }
     },
     "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
   }
   ```

3. **Clean starter demo boilerplate**:
   - Remove the template demo screens, components, web CSS, demo images, and one-off reset script:
     ```bash
     rm -rf src scripts \
       assets/images/expo-badge.png \
       assets/images/expo-badge-white.png \
       assets/images/expo-logo.png \
       assets/images/logo-glow.png \
       assets/images/react-logo.png \
       assets/images/react-logo@2x.png \
       assets/images/react-logo@3x.png \
       assets/images/tutorial-web.png \
       assets/images/tabIcons \
       assets/images/favicon.png
     ```
   - Create a minimal root layout at `src/app/_layout.tsx`:
     ```tsx
     import { Stack } from "expo-router";
     import { SafeAreaProvider } from "react-native-safe-area-context";

     export default function RootLayout() {
       return (
         <SafeAreaProvider>
           <Stack screenOptions={{ headerShown: false }} />
         </SafeAreaProvider>
       );
     }
     ```
   - Create a minimal initial screen at `src/app/index.tsx` using `StyleSheet.create` and `SafeAreaView`:
     ```tsx
     import { StyleSheet, Text } from "react-native";
     import { SafeAreaView } from "react-native-safe-area-context";

     export default function HomeScreen() {
       return (
         <SafeAreaView style={styles.container}>
           <Text style={styles.title}>Ready</Text>
         </SafeAreaView>
       );
     }

     const styles = StyleSheet.create({
       container: {
         flex: 1,
         alignItems: "center",
         justifyContent: "center",
         padding: 24,
       },
       title: {
         fontSize: 24,
         fontWeight: "600",
       },
     });
     ```

4. **Update `package.json` scripts**:
   Remove `web` and `reset-project` and add the standard verification, formatting, build, and health-check scripts:

   ```json
   {
     "scripts": {
       "start": "expo start",
       "ios": "expo start --ios",
       "android": "expo start --android",
       "typecheck": "expo customize tsconfig.json && tsc --noEmit",
       "lint": "expo lint",
       "lint:fix": "expo lint --fix",
       "format": "prettier --write .",
       "format:check": "prettier --check .",
       "build": "expo export --platform ios --platform android",
       "doctor": "npx --yes expo-doctor",
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

Run the full verification gate and Expo health check, confirming exit code `0` with zero errors and zero lint warnings:

```bash
npm run check
npm run doctor
```

(`npm run check` runs `npm run typecheck && npm run lint && npm run format:check && npm run build`, where `typecheck` regenerates Expo Router typed routes before `tsc --noEmit` and `build` bundles both iOS and Android JS/assets via Metro without requiring Xcode or Android Studio.)

Once both pass cleanly, inspect `git status` (note that `.expo/`, `dist/`, `expo-env.d.ts`, `/ios`, and `/android` are gitignored) and create the bootstrap commit:

```bash
git add -A
git commit -m "chore: bootstrap expo project"
```
