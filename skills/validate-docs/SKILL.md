---
name: validate-docs
description: Fact-check documentation against the codebase. Extracts technical claims (file paths, env vars, packages, commands, routes, enums, tables/columns) from markdown via regex, verifies each via filesystem stat, literal grep, or manifest lookup, and reports only missing claims. Token-conscious — skips prose, function signatures, and type shapes by default. Use when user wants to validate docs, check docs accuracy, fact-check documentation, verify doc claims, spot hallucinated paths/enums/routes in docs/agent/, docs/handbook/, AGENTS.md, or any markdown folder. Also triggers on "/validate-docs", "check if the docs are correct", "audit docs for hallucinations".
---

# Validate Docs

Read-only validator. No edits, no persistence — terminal report only. Catches the common hallucination mode where generated docs reference file paths, enum values, routes, or env vars that do not exist in the actual codebase.

## Phase 1: Choose target

Probe the repo for known doc folders. Present to user:

```
Detected:
  1. docs/agent/
  2. docs/handbook/
  3. Other (I'll ask for a path)

Which to validate? [1/2/3]
```

- Omit lines whose folder does not exist.
- If none detected, prompt directly for a path.
- Single target per run.

Accept `--deep` flag. Default checks skip function signatures, type shapes, and arbitrary prose. `--deep` enables them but still uses grep-only verification (no parsing).

## Phase 2: Extract claims

Walk every `*.md` under the target path. For each file, scan line-by-line. Capture `(file, line, claim_type, subject)`. Regex patterns:

| Claim type | Pattern | Notes |
|------------|---------|-------|
| Path | `` `([^`]*\/[^`]+\.[a-z0-9]+)` `` | Backticked, contains `/`, has extension |
| Env var | `` `([A-Z][A-Z0-9_]{2,})` `` | All-caps snake; ≥3 chars; only count when heading/surrounding text mentions `env`, `environment`, `.env` |
| Package | `` `([@a-z][@a-z0-9-/.]{1,})` `` | Backticked lowercase token; cross-checked against manifest deps |
| Command | `\b(npm\|pnpm\|yarn\|bun\|cargo\|make\|just\|task\|go)\s+([a-z][a-z0-9:-]*)` | The second capture is the script/target name |
| Route | `\b(GET\|POST\|PUT\|PATCH\|DELETE)\s+(\/[a-zA-Z0-9:/_\-{}.]*)` | HTTP verb + path |
| Enum value | Backticked identifier within 2 lines of `enum`, `status`, `role`, `values`, `one of` | |
| Table/column | Backticked identifier within 2 lines of `table`, `column`, `field`, `schema` | |

Prose that matches none of these yields zero claims. Features, design-decisions, glossary, contributing, getting-started pages are scanned the same way; they naturally return few or no claims.

### `--deep` extensions

- Function signature: `` `([a-z][a-zA-Z0-9_]*)\s*\(` `` — name captured
- Type/interface: backticked PascalCase token near `interface`, `type`, `class`
- Prose factual claim: backticked identifier not caught by any rule above — generic fallback

## Phase 3: Verify

Three primitives, no framework-specific parsers.

### 3a. Filesystem stat (paths)

`test -e <claim>` relative to repo root. Present if the file or directory exists.

### 3b. Manifest parse (packages, commands, env vars)

Read once per run, cache in memory.

- **Packages**: `package.json`:dependencies+devDependencies, `Cargo.toml`:dependencies, `pyproject.toml`:dependencies, `go.mod`:require, `composer.json`:require, `Gemfile`:gem lines.
- **Commands**: `package.json`:scripts, `Makefile` targets, `justfile` recipes, `Taskfile.yml`:tasks.
- **Env vars**: `.env.example`, `.env.*.example`. Never read `.env` values.

Claim present if the name is a key in the relevant manifest.

### 3c. Literal grep (routes, enums, tables, columns, plus `--deep` types)

Use ripgrep via the Grep tool. Search the exact subject as a literal string.

Exclude paths:
- `node_modules/`, `.git/`, `dist/`, `build/`, `.next/`, `.nuxt/`, `.turbo/`, `target/`, `.vitepress/cache/`, `.vitepress/dist/`
- **The target doc folder itself** (prevents the doc from citing itself as evidence)

Include everything else, including other markdown folders — cross-doc mentions count as evidence.

Claim present if ≥1 match found.

## Phase 4: Report

Console only. Two verdicts: valid (silent), missing (reported).

Format:

```
Validated: <target-path> — <N> files, <M> claims checked, <K> missing.

<doc-file-1>
  <line> — <subject> — <reason>
  <line> — <subject> — <reason>

<doc-file-2>
  <line> — <subject> — <reason>
```

Reason strings:
- Paths: `file does not exist`
- Packages: `not in <manifest-file> deps`
- Commands: `not in <manifest-file> scripts` (or `Makefile`, `justfile`, etc.)
- Env vars: `not in .env.example`
- Routes / enums / tables / columns: `no literal match in source`

If `K == 0`, print only the summary line and exit.

If target path has no `.md` files, print: `No markdown files under <target-path>.` and exit.

## Rules

- Never modify docs. Never write files. Terminal output only.
- Never read secret files: `.env`, `.env.local`, `.env.production`. Only `.env.example` variants.
- Never invoke LLM-based extraction. Regex + stat + grep only.
- When a claim could match multiple types (e.g., a command that looks like a path), pick the most specific pattern — path beats command, route beats path.
- Deduplicate claims per file: same subject repeated on multiple lines reports each line separately; same subject on same line reports once.
- Cap per-run work at 500 claims. If exceeded, print a warning and truncate — user should narrow the target path.
