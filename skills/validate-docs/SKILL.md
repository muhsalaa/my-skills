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

| Claim type   | Pattern                                                                              | Notes                                                                                                                                                             |
| ------------ | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Path         | `` `([^`]*\/[^`]+\.[a-z0-9]+)` ``                                                    | Backticked, contains `/`, has extension. **Reject** URLs (`scheme://`), placeholders and globs (`*`, `<`, `{`, `NNNN`, `...`), and anything containing whitespace |
| Env var      | `` `([A-Z][A-Z0-9_]{2,})` ``                                                         | All-caps snake; ≥3 chars; only count when heading/surrounding text mentions `env`, `environment`, `.env`                                                          |
| Package      | `` `([@a-z][@a-z0-9-/.]{1,})` ``                                                     | Low signal — see 3b. Require `@scope/name` or a hyphen, never a token ending in a code extension, never something already caught as a path                        |
| Command      | `\b(npm\|pnpm\|yarn\|bun\|cargo\|make\|just\|task\|go)\s+([^\s`]+)(?:\s+([^\s`]+))?` | Skip modifier subcommands (`run`, `x`, `dlx`, `exec`, `create`, `add`, `install`, `pm`, `ci`) and take the next token; skip `--flags` and absolute paths          |
| Route        | `\b(GET\|POST\|PUT\|PATCH\|DELETE)\s+(\/[a-zA-Z0-9:/_\-{}.]*)`                       | HTTP verb + path                                                                                                                                                  |
| Enum value   | Backticked identifier within 2 lines of `enum`, `status`, `role`, `values`, `one of` |                                                                                                                                                                   |
| Table/column | Backticked identifier within 2 lines of `table`, `column`, `field`, `schema`         |                                                                                                                                                                   |

**Tuned rules — apply these or the report drowns in false positives.** Learned from a real run over `docs/agent/`, where ~280 of 286 findings were the validator's own fault:

- A path beats a package. Backticked `src/container.ts` contains `/` and reads as a "package"; the path pattern wins, and a package claim must never match a token ending in a code extension (`.ts`, `.tsx`, `.js`, `.json`, `.sql`, `.sh`, `.yml`, `.md`).
- A command's target may be a file, not a script name: `bun run scripts/reset-db.ts` is verified with `test -e`, not against `package.json` scripts.
- Prose looks like a command (`make commands`, `bun install`) — skip tokens that are not script names in any manifest.
- Keep the target doc folder out of grep evidence, but include other markdown folders: cross-doc mentions are evidence.

Prose that matches none of these yields zero claims. Features, design-decisions, glossary, contributing, getting-started pages are scanned the same way; they naturally return few or no claims.

### `--deep` extensions

- Function signature: `` `([a-z][a-zA-Z0-9_]*)\s*\(` `` — name captured
- Type/interface: backticked PascalCase token near `interface`, `type`, `class`
- Prose factual claim: backticked identifier not caught by any rule above — generic fallback

## Phase 3: Verify

Three primitives, no framework-specific parsers.

### 3a. Filesystem stat (paths)

`test -e <claim>` relative to repo root. Present if the file or directory exists.

Docs often quote a path relative to `src/` or to the feature folder. Before reporting a miss, retry with a `src/` prefix and then as a suffix match against every file in the repo. Only report when all three attempts fail.

### 3b. Manifest parse (packages, commands, env vars)

Read once per run, cache in memory.

- **Packages**: `package.json`:dependencies+devDependencies, `Cargo.toml`:dependencies, `pyproject.toml`:dependencies, `go.mod`:require, `composer.json`:require, `Gemfile`:gem lines.
- **Commands**: `package.json`:scripts, `Makefile` targets, `justfile` recipes, `Taskfile.yml`:tasks.
- **Env vars**: `.env.example`, `.env.*.example`. Never read `.env` values.

Claim present if the name is a key in the relevant manifest.

**Package claims are the weakest signal — skip them by default.** The same token shape covers filenames (`magic-link.ts`), container names (`familya-postgres-dev`), Docker images (`oven/bun`), GitHub Actions (`oven-sh/setup-bun@v2`), subpath imports (`hono/jsx`) and git hook names (`pre-commit`). Report a package only when the line also mentions install/package/dependency and the token resolves nowhere in the codebase.

### 3c. Literal grep (routes, enums, tables, columns, plus `--deep` types)

Use ripgrep via the Grep tool. Search the exact subject as a literal string.

Exclude paths:

- `node_modules/`, `.git/`, `dist/`, `build/`, `.next/`, `.nuxt/`, `.turbo/`, `target/`, `.vitepress/cache/`, `.vitepress/dist/`
- **The target doc folder itself** (prevents the doc from citing itself as evidence)

Include everything else, including other markdown folders — cross-doc mentions count as evidence.

Claim present if ≥1 match found. Before reporting env vars, enum values, table names and column names, retry the search case-insensitively (`PG_CLIENT_KEY` vs `pg_client_key`, `SUBSCRIPTION_STATUS.ACTIVE` vs the enum's own spelling) and report only when both attempts fail.

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
