# CLAUDE.md Generation Rules

This document defines the structure, line budget, and complementarity contract for generating CLAUDE.md.

## Purpose

CLAUDE.md is auto-loaded into every Claude Code conversation. It must be:

- **Slim:** ~80 lines (hard cap: 100 lines). Every line costs tokens on every interaction.
- **Operational:** Commands, imports, env vars -- things needed immediately, every session.
- **A pointer:** It directs the reader to AGENTS.md for everything else.

## Required Sections

### 1. Header + AGENTS.md Pointer (3-5 lines)

```markdown
# CLAUDE.md

{One-sentence project description.}

For architecture, module rules, coding conventions, and tech stack details, see **[AGENTS.md](./AGENTS.md)** and its per-module children. Read AGENTS.md **Golden Rules** before any non-trivial task (Immutable Constraints, Do's/Don'ts).
```

This pointer is MANDATORY. It is the bridge between the two files.

### 2. Common Commands (~20-25 lines)

```markdown
## Common Commands

{Fenced bash block with grouped commands: dev, build, lint, test, database.}
{Include actual flags. Group with comments.}
```

This is the ONE section allowed to overlap with AGENTS.md Operational Commands.

### 3. Critical Imports (~10 lines)

```markdown
## {Domain} Imports

{Fenced TypeScript block showing the most commonly needed import patterns.}
{Only include imports that developers need to look up frequently.}
{Add inline comments explaining when to use each.}
```

Keep this to the top 3-5 most important import patterns. AGENTS.md has the full list.

### 4. Git (3-5 lines)

```markdown
## Git

- **Main branch:** `{branch}` (not main/master)
- **Commits:** {format description}
```

Summary only. Full git strategy lives in AGENTS.md Standards & References.

### 5. Environment Variables (5-10 lines)

```markdown
## Environment Variables

{Variable names grouped by target, one line each.}
{No descriptions -- just names. AGENTS.md has descriptions.}
```

List variable names only. AGENTS.md provides the detailed descriptions.

## Prohibited Content in CLAUDE.md

The following MUST NOT appear in CLAUDE.md (they belong exclusively in AGENTS.md):

- Architecture diagrams or monorepo layout trees
- Golden Rules (Immutable Constraints, Do's, Don'ts)
- Implementation patterns or code style guides
- Tech stack summary tables
- Route structure or file naming conventions
- Testing strategy details (beyond test commands)
- Context Map (action-based routing)
- Standards & References section
- Maintenance Policy

## Complementarity Contract

This contract defines the ownership boundary between CLAUDE.md and AGENTS.md.

### Exclusive to CLAUDE.md

- AGENTS.md pointer paragraph (the bridge text)

### Exclusive to AGENTS.md

- Project Context (full description, business goals)
- Tech Stack Summary table
- Golden Rules (all subsections)
- Standards & References (conventions, git strategy, maintenance policy)
- Context Map (action-based routing)
- All nested module details

### Allowed Overlap (Operational Commands only)

Operational Commands appear in BOTH files because they are needed instantly every session. The content should be identical or near-identical to avoid confusion.

### Summary-Detail Relationship (not duplication)

These items appear in both files but at different levels of detail:

| Item     | CLAUDE.md                             | AGENTS.md                                 |
| -------- | ------------------------------------- | ----------------------------------------- |
| Imports  | Top 3-5 patterns with inline comments | Full import catalog with usage rules      |
| Env vars | Variable names only, grouped          | Names + descriptions + constraints        |
| Git      | Branch name + commit format one-liner | Full strategy, CI pipeline, branch naming |
| Testing  | Commands only                         | Strategy, patterns, coverage rules        |

This is NOT duplication. CLAUDE.md provides the "cheat sheet"; AGENTS.md provides the "manual page".

## Generation Process

1. Read the AGENTS.md hierarchy (generated in Phase 1 or pre-existing).
2. Extract only the items listed in "Required Sections" above.
3. Compress each item to fit the line budget.
4. Verify no prohibited content leaked in.
5. Count lines. If over 100, cut the least essential items (starting from env vars descriptions, then command variations).

## Custom Block Handling

If the existing CLAUDE.md contains `<!-- custom -->` ... `<!-- /custom -->` blocks:

1. Extract the custom block content.
2. Insert it at the same relative position in the regenerated file.
3. The custom block's lines count toward the 100-line cap but should be accommodated by trimming generated content if needed.

If no existing CLAUDE.md or no custom markers, append this at the end:

```markdown
<!-- custom -->
<!-- Add project-specific notes below. This block is preserved across regeneration. -->
<!-- /custom -->
```
