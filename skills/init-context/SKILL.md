---
name: init-context
description: "Generate complementary CLAUDE.md and AGENTS.md for the project. Use when initializing or refreshing AI context files."
disable-model-invocation: true
argument-hint: "[all | agents | claude | <path>]"
---

# Role

You are a **Principal Architect for AI Context & Governance**. You analyze the user's project and produce a complementary pair of context files:

- **CLAUDE.md** -- Slim operational reference (~80 lines), auto-loaded every session.
- **AGENTS.md hierarchy** -- Detailed governance, architecture, and module-specific rules.

These two layers form a single system. CLAUDE.md is the "quick card"; AGENTS.md is the "full manual". They must never duplicate each other except where explicitly permitted by the Complementarity Rules in `claude-prompt.md`.

# Scope

Parse `$ARGUMENTS` to determine scope:

| Argument         | Behavior                                                           |
| ---------------- | ------------------------------------------------------------------ |
| (empty) or `all` | Run all phases (Discovery -> AGENTS.md -> CLAUDE.md -> Verify)     |
| `agents`         | Phase 0 + Phase 1 only (AGENTS.md hierarchy)                       |
| `claude`         | Phase 0 + Phase 2 only (CLAUDE.md)                                 |
| `<path>`         | Phase 0 (scoped) + generate/refresh nested AGENTS.md at given path |

# Workflow

## Phase 0: Discovery

Perform comprehensive project analysis before generating any file. Gather **facts only** -- no assumptions.

**Token budget guidance:** For large monorepos (5+ workspace members), use the Agent tool with `subagent_type=Explore` to parallelize structure/code scanning (steps 0.1-0.3). This keeps the main context window lean. For small projects, inline Glob/Read/Grep is fine.

### 0.1 Structure Scan

Read and analyze (use Glob + Read, not Bash):

- `package.json` (root + all workspace members) -- deps, scripts, engines
- `tsconfig.json` / `tsconfig.*.json` -- compiler options, paths aliases
- Framework configs: `next.config.*`, `vite.config.*`, `drizzle.config.*`, `tailwind.config.*`, `postcss.config.*`
- `.env.example` or `.env.local.example` -- env var inventory
- Lockfile presence: `pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`, `bun.lockb`
- Monorepo config: `turbo.json`, `pnpm-workspace.yaml`, `lerna.json`

### 0.2 Documentation Scan

Read existing context sources (skip if absent):

- `README.md` (root + workspace members)
- `.cursor/rules/`, `.github/copilot-instructions.md`, `.windsurfrules`
- Existing `CLAUDE.md` and any `AGENTS.md` files
- `CONTRIBUTING.md`, `docs/` folder

### 0.3 Code Pattern Inference

Use Grep + Read on a sample of source files (max 10-15 files across the project):

- Import patterns: path aliases, barrel exports, relative vs absolute
- Naming conventions: file names, component names, function names
- State management: Redux, Zustand, Context, TanStack Query, signals
- Styling: CSS Modules, Tailwind, styled-components, CSS-in-JS
- Data access: ORM usage, client patterns, API route conventions
- Testing patterns: test file locations, naming, frameworks used

### 0.4 Git Analysis

```bash
git log --oneline -20                    # Recent commit style
git branch -a | head -20                 # Branch strategy
git remote -v                            # Remote setup
```

Detect: main branch name, commit convention (Conventional Commits?), branch naming pattern.

### 0.5 Existing Context Audit

If CLAUDE.md or AGENTS.md already exist:

1. Read them fully.
2. Identify `<!-- custom -->` ... `<!-- /custom -->` marker blocks -- contents inside these markers MUST be preserved verbatim in regenerated files.
3. Note their current line counts for the verification phase.

### 0.6 Boundary Detection

Identify nested AGENTS.md candidates by checking for:

- **Dependency Boundary:** Separate `package.json` / `requirements.txt` / `Cargo.toml`
- **Framework Boundary:** Tech stack transition points (e.g., Next.js app vs Deno edge functions)
- **Logical Boundary:** High-density business logic modules, separate deployment targets

Record each boundary as: `{ path, type, reason, key_deps }`.

## Phase 1: Generate AGENTS.md Hierarchy

Reference: `agents-prompt.md` for all section templates and rules.

Order of operations:

1. Generate root `./AGENTS.md` first.
2. Generate each nested `AGENTS.md` in dependency order (leaves first, then parents that reference them).
3. Ensure the root Context Map links to every nested file.

For each file:

- If existing file has `<!-- custom -->` blocks, preserve them at their original positions.
- If existing file has NO custom markers, warn the user before overwriting: "Existing {path} has no custom markers. Contents will be replaced."
- Append a custom marker block at the end of every generated file:

```markdown
<!-- custom -->
<!-- Add project-specific notes below. This block is preserved across regeneration. -->
<!-- /custom -->
```

## Phase 2: Generate CLAUDE.md

Reference: `claude-prompt.md` for section templates, line budget, and complementarity rules.

1. Read the AGENTS.md generated in Phase 1 (or existing if scope is `claude` only).
2. Extract only what belongs in CLAUDE.md per the complementarity contract.
3. Generate slim CLAUDE.md (~80 lines).
4. Preserve any `<!-- custom -->` blocks from existing CLAUDE.md.
5. Append custom marker block if not already present.

## Phase 3: Verification

Run these checks using the specified tools and report results:

1. **Line Count:** Count lines of each generated file (use Bash `wc -l`). CLAUDE.md <= 100 lines. Each AGENTS.md <= 500 lines.
2. **Complementarity Check:** Read both CLAUDE.md and root AGENTS.md. Manually compare section headings. Only `## Common Commands` / `## Operational Commands` may share content. Flag any other heading that appears in both.
3. **Link Integrity:** For each path in AGENTS.md Context Map, use Glob to confirm the target file exists.
4. **Custom Block Integrity:** If input files had `<!-- custom -->` blocks, use Grep to confirm `<!-- custom -->` and `<!-- /custom -->` appear in each output file.
5. **No Emojis:** Use Grep with pattern `[\x{1F300}-\x{1F9FF}]` on generated files (best effort; visually scan if grep misses).

Report format:

```
Verification Results:
- CLAUDE.md: {n} lines [PASS/FAIL]
- AGENTS.md (root): {n} lines [PASS/FAIL]
- AGENTS.md (nested): {path}: {n} lines [PASS/FAIL] (for each)
- Complementarity: [PASS/FAIL] {details if fail}
- Link Integrity: [PASS/FAIL] {broken links if fail}
- Custom Blocks: [PASS/FAIL] {details if fail}
- Emoji Check: [PASS/FAIL]
```

# Execution Rules

1. **Direct Execution:** Do not ask "should I create this file?" -- generate immediately.
2. **Markdown Only:** Output valid Markdown. No prose explanations inside the generated files.
3. **Korean Content:** If the project's user-facing strings are in Korean, write AGENTS.md rule descriptions in English but note the Korean-string requirement in Golden Rules.
4. **Facts Over Assumptions:** Every rule you write must be traceable to a discovery finding. If uncertain, add a `TODO:` comment rather than guessing.
