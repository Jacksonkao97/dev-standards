# dev-standards

Personal engineering standards, coding guidelines, and reproducible project bootstrap recipes built on a **Recipe + Version-Safe Overlay** architecture.

Rather than maintaining static starter repositories with pinned `package.json` and lockfiles that quickly go stale, `dev-standards` pairs:

1. **Official upstream scaffolding** (e.g., `create-next-app@latest`) to pull fresh, compatible framework defaults.
2. **Version-safe overlay files** (`AGENTS.md`, `CLAUDE.md`, linter/formatter/editor configs) copied cleanly on top.
3. **A Mandatory Freshness & Breaking-Change Check** before scaffolding or coding, so agents catch changed CLI flags, deprecated options, or breaking API changes early and offer to update this repo.

---

## Repository Structure

```text
dev-standards/
├── README.md
├── .gitignore
├── common/
│   └── BASE_RULES.md                 # Universal rules across all frameworks
└── stacks/
    └── nextjs/
        ├── RULES.md                  # Next.js 16+ coding style & architecture guide
        ├── BOOTSTRAP.md              # Step-by-step reproducible bootstrap recipe
        └── files/                    # Version-safe config & agent overlays
            ├── AGENTS.md             # Tool-agnostic project rules (~120–160 lines)
            ├── CLAUDE.md             # Pointer file (@AGENTS.md)
            ├── .oxlintrc.json        # Native Rust Oxlint configuration
            ├── .prettierrc           # Prettier + Tailwind CSS v4 sorting
            ├── .prettierignore
            └── .editorconfig
```

---

## Stack Catalog

| Stack | Status | Core Defaults | Guides |
| --- | --- | --- | --- |
| `nextjs` | Active | Next.js 16+ App Router, TypeScript strict (`noUncheckedIndexedAccess`), Turbopack, Tailwind CSS v4, `lucide-react`, `zod`, `server-only`, Oxlint, Prettier | [`RULES.md`](stacks/nextjs/RULES.md) · [`BOOTSTRAP.md`](stacks/nextjs/BOOTSTRAP.md) |

---

## Usage Across AI Coding Tools

Every stack overlay centers on a plain, tool-agnostic **`AGENTS.md`** file with no vendor-specific frontmatter, paired with lightweight pointers where tools expect them.

### Antigravity

- Attach or reference the saved `dev-standards` cloud pointer Skill.
- When creating a new project or applying your standards, Antigravity fetches `common/BASE_RULES.md` and `stacks/<framework>/{RULES.md,BOOTSTRAP.md,files/}` from this repo, performs the Mandatory Freshness & Breaking-Change Check, and follows the bootstrap recipe.
- Inside any bootstrapped project, `AGENTS.md` governs day-to-day coding automatically.

### Claude Code

- Every bootstrapped project includes a one-line `CLAUDE.md` containing `@AGENTS.md`, so Claude Code automatically loads the exact same project rules without duplicating content.
- When starting a new repo with Claude Code, point it to `https://github.com/Jacksonkao97/dev-standards` and ask it to follow `common/BASE_RULES.md` + `stacks/<framework>/BOOTSTRAP.md`.

### Cursor

- Cursor reads `AGENTS.md` directly at the repository root (and respects `.editorconfig`, `.prettierrc`, and `.oxlintrc.json`).
- No separate `.cursorrules` duplication is needed; keep `AGENTS.md` as the single source of truth.

### Codex

- OpenAI Codex reads `AGENTS.md` automatically from the project root and walks parent directories if nested.
- To bootstrap a fresh repo, ask Codex to fetch `common/BASE_RULES.md` and `stacks/<framework>/BOOTSTRAP.md` from `Jacksonkao97/dev-standards`.

---

## Adding a New Stack

To add another framework (for example, `fastapi`, `vite-react`, or `astro`):

1. Create `stacks/<stack-name>/` with three items:
   - `RULES.md` — Stack-specific toolchain, folder architecture, conventions, testing defaults, and Mandatory Freshness & Breaking-Change Check (extending `common/BASE_RULES.md`).
   - `BOOTSTRAP.md` — Reproducible Step 0 (Freshness Check) through final verification command using the official upstream scaffold CLI (no committed `package.json` or lockfile inside `dev-standards`).
   - `files/` — Minimal version-safe overlay files including a concise (~120–160 line) plain-Markdown `AGENTS.md`, `CLAUDE.md` (`@AGENTS.md`), and linter/formatter/editor configs.
2. Dry-run `BOOTSTRAP.md` end-to-end in a temporary directory and verify that all check commands (`typecheck`, `lint`, `format:check`, `build`) pass with exit code `0`.
3. Add the new stack to the **Stack Catalog** table in this `README.md`.
4. Commit with a Conventional Commit message (e.g., `feat(stacks): add <stack-name> stack`).
