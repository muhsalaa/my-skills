---
name: init-docs
description: Initialize human-facing onboarding documentation for a codebase. Scans the project, proposes a docs/handbook/ folder tailored to the stack, interviews the user for rationale and narrative, generates grouped markdown for new engineers (onboarding, contributing, architecture, design decisions, per-feature), and optionally installs a local VitePress reader. Use when user wants to initialize handbook docs, create onboarding documentation, set up docs/handbook/, or says "init docs", "create handbook", "initialize documentation for humans", "onboarding docs".
---

# Init Docs

Generates `docs/handbook/` — a human-readable onboarding handbook for new engineers. Topics mirror agent documentation but the voice, depth, and structure are built for people: narrative, rationale, and grouped navigation.

Optionally installs a local VitePress reader so the handbook renders with sidebar navigation and offline search. Local only; no deployment config.

## Self-contained

This skill is standalone. It has its own scan logic, catalog, interview rules, and generator. It may overlap with sibling skills such as `init-agent-docs` (which targets machine readers rather than humans), but it does not depend on them.

Output-layer cross-links are the only coupling: when `docs/agent/` or `AGENTS.md` is detected in the project, the skill generates a `for-llms.md` page in the handbook that points readers at those machine-readable docs.

- Full flow in [WORKFLOW.md](WORKFLOW.md).
- Doc catalog in [CATALOG.md](CATALOG.md).
- VitePress install + config in [VITEPRESS.md](VITEPRESS.md).

## Phases

1. **Scan** — reuse parent's Phase 1 signal detection + bounded context enumeration
2. **Propose** — present signal-driven docs + always-on docs from [CATALOG.md](CATALOG.md); propose detected choices eligible for ADR; user ticks/skips/adds
3. **Interview** — one question per approved doc (headline), plus rationale per ticked ADR; block on unknowns required for downstream decisions
4. **Generate** — write `docs/handbook/` with grouped subfolders, frontmatter, human voice, TODO/TBD markers where user skipped
5. **VitePress** (optional) — ask user consent, pre-flight Node check, write self-contained config, install via detected package manager

## Key Constraints

- **Never modify existing docs** — root `README.md`, `CONTRIBUTING.md`, `docs/agent/`, `AGENTS.md` are read-only inputs. Handbook is new output.
- **Abort on collision** — if `docs/handbook/` already exists, ask user to abort or write to `docs/handbook-<timestamp>/`. Never merge or overwrite.
- **Audience** — competent software engineer, no project context. Don't explain language/framework basics; do explain domain, decisions, and local-setup quirks.
- **One question at a time** in interview. Every question ships with recommended answer and rationale.
- **TODO/TBD markers visible** — when user skips a question, generated prose marks the gap so nothing ships silently empty.
- **Self-contained VitePress** — own `package.json` inside `docs/handbook/`. No pollution of project root.

## Output Structure

```
docs/handbook/
├── README.md            # GitHub-visible: how to run the reader
├── index.md             # VitePress homepage
├── getting-started.md
├── contributing.md
├── glossary.md
├── tech-stack.md  conventions.md  workflows.md  ...   (from parent CATALOG)
├── architecture.md  data-model.md  auth.md  ...       (signal-driven)
├── ci-cd.md  deployment.md  environments.md  ...      (ops, signal-driven)
├── design-decisions/
│   ├── README.md        # index of ADRs
│   ├── template.md
│   └── NNNN-<slug>.md   # one per ticked decision
├── features/            # if bounded contexts detected
│   ├── README.md
│   └── <name>.md
├── for-llms.md          # only if docs/agent/ or AGENTS.md detected
├── .vitepress/          # only if VitePress install accepted
│   └── config.ts
├── package.json         # only if VitePress install accepted
└── .gitignore           # always — covers node_modules, .vitepress/cache, .vitepress/dist
```

See [CATALOG.md](CATALOG.md) for the full doc menu and [VITEPRESS.md](VITEPRESS.md) for reader setup.
