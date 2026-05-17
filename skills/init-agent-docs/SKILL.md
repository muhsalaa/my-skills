---
name: init-agent-docs
description: Initialize agent-friendly documentation for a codebase. Scans the project, detects backend/frontend/monorepo signals, proposes a docs/agent/ folder structure tailored to the stack, interviews the user to validate requirements, generates minimal AGENTS.md with progressive disclosure links, and creates topic-specific markdown files. Use when user wants to initialize agent documentation, create AGENTS.md, set up docs/agent/, or says "init agent docs", "create agent docs", "initialize documentation for agents".
---

# Init Agent Docs

Generates a `docs/agent/` folder and a minimal `AGENTS.md` using progressive disclosure. AGENTS.md stays small — detail lives in `docs/agent/` files loaded only when relevant. The goal is to create whole new set of docs for agents to use.

Works for backend, frontend, full-stack, and monorepo projects. Proposal is signal-driven: scan detects what the project actually uses and proposes matching docs from [CATALOG.md](CATALOG.md).

## Phases

See [WORKFLOW.md](WORKFLOW.md) for full detail. Summary:

1. **Scan** — read codebase, detect stack + signals, enumerate bounded contexts
2. **Propose** — list signal-matched docs from CATALOG.md; user approves/skips/adds
3. **Interview** — validate requirements one question at a time
4. **Generate** — write `docs/agent/*.md` (with subfolders when warranted) + hub `README.md`
5. **AGENTS.md** — write minimal root file with links

## Key Constraints

- AGENTS.md = minimal: 1-sentence description + package manager + key commands + links only
- Never document file paths in AGENTS.md (go stale); document capabilities/concepts
- Answer codebase questions yourself — only ask user what code can't reveal
- Do not proceed past interview phase if user cannot answer a required question
- Every generated doc includes `last_updated: YYYY-MM-DD` in frontmatter
- Propose every fired signal; let user skip rather than self-censoring
- Use subfolders when >6 docs or per-bounded-context docs are approved

## Output Structure

Adapts to project shape. Examples:

**Flat (small)**
```
docs/agent/
├── tech-stack.md
├── conventions.md
└── workflows.md
AGENTS.md
```

**Layered backend (DDD / Clean Arch)**
```
docs/agent/
├── README.md
├── tech-stack.md  conventions.md  workflows.md
├── architecture.md  canonical-shape.md  dependency-rules.md
├── data-model.md  db-migrations.md  auth.md  http-layer.md
├── error-handling.md  cron-jobs.md  testing.md  deployment.md
└── features/
    ├── README.md
    └── [one doc per bounded context]
AGENTS.md
```

**Frontend**
```
docs/agent/
├── README.md
├── tech-stack.md  conventions.md  workflows.md
├── framework.md  routing.md  ui-components.md  styling.md
├── state-management.md  data-fetching.md  forms.md  i18n.md
└── testing.md  build.md
AGENTS.md
```

**Monorepo**
```
docs/agent/
├── README.md  monorepo.md  tech-stack.md  workflows.md
├── apps/    [one doc per app]
└── packages/  [one doc per shared package]
AGENTS.md
```

See [CATALOG.md](CATALOG.md) for the full doc menu + signal-to-doc mapping.
