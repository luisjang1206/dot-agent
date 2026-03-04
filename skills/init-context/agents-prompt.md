# AGENTS.md Generation Rules

This document defines the structure, style, and content rules for generating AGENTS.md files. It inherits the Core Philosophy and Detection Logic from the master prompt and adds precision constraints.

## Core Philosophy

1. **Strict 500-Line Limit:** Every AGENTS.md file stays under 500 lines.
2. **No Emojis:** Never use emoji characters in any AGENTS.md file.
3. **Central Control & Delegation:** Root AGENTS.md is the "control tower". Module details are delegated to nested files.
4. **Machine-Readable Clarity:** Provide actionable directives (Golden Rules, Operational Commands), not advisory prose.

## Writing Style

- Imperative mood: "Use X", "Never do Y" -- not "You should consider using X".
- Paragraphs: 3 sentences max. Prefer bullet lists.
- Code blocks: Use fenced code blocks for commands, file paths, and patterns.
- No filler: Remove words like "basically", "simply", "just", "please note that".
- Headings: ATX-style (`##`, `###`). Max depth `####`.

## Root AGENTS.md -- Required Sections

### 1. Project Context

```markdown
## Project Context

{One-paragraph project description: what it is, who it's for, primary business goal.}

### Tech Stack Summary

| Layer | Technology |
| ----- | ---------- |
| ...   | ...        |

### Monorepo Layout (if applicable)

{Tree diagram of top-level directories with one-line descriptions.}
```

Content source: Phase 0 structure scan + package.json analysis.

### 2. Operational Commands

```markdown
## Operational Commands

{Grouped by category: Development, Build & Quality, Testing, Database, Deployment.}
{Use fenced bash code blocks. Include actual flags and common variations.}
```

This section is the ONE allowed overlap with CLAUDE.md (both files include it for instant access).

### 3. Golden Rules

```markdown
## Golden Rules

### Immutable Constraints

{Numbered list. Security and architecture rules that must never be violated.}
{Examples: "Never hardcode secrets", "Never use service role key client-side".}

### Do's

{Bulleted list. Positive behavior patterns.}
{Each item starts with "Always..." or "Use..." or a direct verb.}

### Don'ts

{Bulleted list. Anti-patterns to avoid.}
{Each item starts with "Do not..." or "Never...".}
```

Content source: Code pattern inference + framework constraints + security analysis.

### 4. Standards & References

```markdown
## Standards & References

### Coding Conventions

{Formatter config, naming conventions, export patterns, import rules.}
{Link to existing style guides if they exist.}

### Git Strategy

{Main branch name, commit format, branch naming, CI pipeline summary.}

### Maintenance Policy

When rules in any AGENTS.md diverge from actual code behavior, update the AGENTS.md to reflect reality and flag the divergence in a commit message.
```

### 5. Environment Variables

```markdown
## Environment Variables

{Grouped by deployment target or application.}
{List variable names with one-line descriptions.}
{Never include actual values.}
```

### 6. Context Map

```markdown
## Context Map (Action-Based Routing)

- **[Trigger/Area Description](./relative/path/AGENTS.md)** -- When to read this file
- ...
```

Constraints:

- No table format. Use bulleted list only.
- No emojis.
- Format: `- **[Label](path)** -- One-line trigger description`
- Every entry must link to a file that exists or is being generated in this run.

## Nested AGENTS.md -- Required Sections

Each nested file covers a single boundary (dependency, framework, or logical).

### 1. Module Context

```markdown
## Module Context

{What this module does, its role in the system, key dependencies.}
{Upstream/downstream relationships with other modules.}
```

### 2. Tech Stack & Constraints

```markdown
## Tech Stack & Constraints

{Libraries and versions specific to this module.}
{Framework constraints: "This folder uses X, not Y".}
{Runtime requirements if different from root.}
```

### 3. Implementation Patterns

```markdown
## Implementation Patterns

{Code patterns used in this module: data access, component structure, error handling.}
{File naming conventions specific to this module.}
{Boilerplate references: "New features follow the pattern in src/features/example/".}
```

Use fenced code blocks for pattern examples. Keep examples minimal (10-15 lines max).

### 4. Testing Strategy

```markdown
## Testing Strategy

{Test framework and runner for this module.}
{Test file location convention.}
{Key test commands specific to this module.}
{Coverage requirements if any.}
```

### 5. Local Golden Rules

```markdown
## Local Golden Rules

{Do's and Don'ts specific to this module.}
{Common mistakes developers make in this area.}
{These supplement, never contradict, root Golden Rules.}
```

## Boundary Detection Logic

Generate a nested AGENTS.md when ANY of these signals are present:

| Signal Type         | Detection Method                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Dependency Boundary | Separate `package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`                                                 |
| Framework Boundary  | Different runtime/framework from parent (e.g., Deno vs Node, Next.js vs Express)                                    |
| Logical Boundary    | Directory contains 10+ source files with distinct domain logic, separate deployment config, or dedicated test suite |

Do NOT generate nested files for:

- Pure configuration directories (`config/`, `.github/`)
- Directories with fewer than 5 source files and no unique constraints
- Shared utility packages with no domain-specific rules beyond what root covers

## Content Quality Checks

Before finalizing any AGENTS.md:

1. Every rule must be traceable to a discovery finding (Phase 0).
2. No rule contradicts another rule in the same file or parent file.
3. No section is empty. If a section has no content, omit it entirely.
4. Code examples compile/run conceptually (no pseudo-syntax).
5. File paths in examples are real paths found during discovery.
