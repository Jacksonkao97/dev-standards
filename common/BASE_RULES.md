# Base Engineering Standards

Universal engineering rules that apply across every repository, language, and framework. Stack-specific guides in `stacks/<framework>/RULES.md` extend these rules—never weaken them.

---

## 1. Mandatory Freshness & Breaking-Change Check (Pre-Coding Gate)

> **STOP & VERIFY BEFORE SCAFFOLDING OR CODING.**

Frameworks, CLIs, linters, and libraries evolve faster than static docs. Before running bootstrap commands or writing feature code:

1. **Check the actual environment and versions**: inspect installed versions (e.g., `package.json`, lockfile, CLI `--version`/`--help`, bundled package docs) and verify against the latest official documentation or release notes when starting a project or upgrading dependencies.
2. **Detect drift or breaking changes**: check whether any CLI flag, config option, API, default behavior, or recommended pattern in these standards has changed, been deprecated, or conflicts/breaks with the target framework version.
3. **If anything changed, deprecated, or conflicts/breaks**:
   - **STOP before coding.** Do not silently work around breaking changes or ignore warnings.
   - Explain clearly **what changed** (old rule/flag/API vs. current behavior/docs).
   - Give a **concrete recommendation** for the current project.
   - **Ask the user how to proceed**, and explicitly offer to update `dev-standards` so future projects stay accurate.

---

## 2. Code Quality & Modularity

- **Keep modules small and focused**: each file, component, or function should have one clear responsibility. Extract helpers when logic becomes nested or repeated.
- **Explicit error handling**: never swallow errors with empty `catch` blocks. Validate assumptions early, return typed errors or raise informative errors at boundaries, and provide clear failure states to callers/users.
- **No dead code or debug leftovers**: remove unused imports, variables, commented-out code blocks, TODOs without context, and stray `console.log` statements before finishing.
- **Prefer clarity over cleverness**: write direct, readable code with descriptive names. Avoid premature abstractions, speculative configurability, and unnecessary dependencies.
- **Preserve existing documentation**: keep unrelated comments, docstrings, and behavioral contracts intact unless intentionally changing them.

---

## 3. Security Hygiene

- **Never commit secrets**: never commit `.env`, `.env.*`, API keys, tokens, credentials, private keys, or service-account files. Provide `.env.example` with placeholder keys only when helpful.
- **Validate all external input at boundaries**: treat request bodies, query strings, URL params, headers, form submissions, webhooks, file uploads, and environment variables as untrusted. Parse and validate with a schema library before use.
- **Isolate server secrets from client bundles**: never expose server-only credentials, database clients, or private helpers to client-side code. Enforce separation with build-time guards (e.g., `server-only`) and strict public env var prefixes.
- **Least privilege & safe defaults**: check authentication and authorization on the server before reading or mutating protected data—never trust client-supplied user IDs, roles, or flags.

---

## 4. Git Workflow

- **Conventional Commits**: use concise, descriptive commit messages following `<type>(<optional-scope>): <summary>` (imperative mood, lowercase summary, no trailing period):
  - `feat:` — new user-facing feature or capability
  - `fix:` — bug fix
  - `refactor:` — code restructuring with no behavior change
  - `docs:` — documentation-only changes
  - `chore:` — tooling, dependencies, configuration, scaffolding
  - `test:` — adding or updating tests
- **Focused, atomic commits**: keep each commit scoped to one logical change. Do not mix unrelated refactors, formatting churn, and feature work in the same commit.
- **Clean working tree**: review `git diff` and `git status` before committing to ensure no unintended files, secrets, or debug logs are staged.

---

## 5. Definition of Done

Never claim a task is complete until all applicable verification checks pass with zero errors:

1. **`typecheck`** — static type checking passes cleanly.
2. **`lint`** — linter passes with zero errors (and no newly introduced warnings without justification).
3. **`format:check`** — code formatting matches project rules.
4. **`build`** — production build succeeds cleanly.
5. **Tests** — all existing unit, integration, and E2E tests pass (plus new tests for bug fixes or critical logic when a test suite exists).

If a check cannot be run or fails, report the exact failure honestly—never skip checks or claim success without verifying.
