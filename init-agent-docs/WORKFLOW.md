# Init Agent Docs — Workflow

Works for **backend, frontend, full-stack, and monorepo** projects. Doc catalog in [CATALOG.md](CATALOG.md) drives proposals; scan detects signals that map to catalog entries.

## Phase 1: Scan

Scan the entire codebase before talking to the user. Produce a **signal inventory**, not just a summary.

### 1a. Baseline (always)

- **Stack**: languages, frameworks, runtime (check `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `Gemfile`, `composer.json`, `mix.exs`, etc.)
- **Package manager**: npm/pnpm/yarn/bun/cargo/pip/poetry/uv/go/etc.
- **Scripts / commands**: `package.json` scripts, Makefile, justfile, Taskfile
- **Existing docs**: `README.md`, `docs/`, `.github/`, wiki, any `.md` files in repo root
- **Linter/formatter configs**: biome/eslint/prettier/rustfmt/ruff/black/golangci → informs conventions doc
- **Folder tree**: top-level + one level deep in `src/` (or equivalent)

### 1b. Signal detection

For each candidate doc in [CATALOG.md](CATALOG.md), check whether its signal fires. Record matches. Common signal classes to check:

**Project shape**
- Monorepo: `apps/` + `packages/`, `turbo.json`, `nx.json`, `pnpm-workspace.yaml`, `lerna.json`, Cargo workspace
- Modular / DDD: `features/`, `modules/`, `contexts/`, `bounded-contexts/`
- Layered: `domain/`, `application/`, `infrastructure/`, `core/`, `adapters/`, `ports/`, `usecases/`

**Backend signals** (dependency + folder combined)
- ORM: drizzle-orm, prisma, typeorm, sqlalchemy, ecto, active_record
- Auth: better-auth, next-auth, clerk, auth0, passport, devise
- HTTP framework: hono, express, fastify, nestjs, koa, axum, gin, fastapi, rails, django
- Jobs: croner, node-cron, bullmq, bee-queue, sidekiq, celery, rq
- Infra: Dockerfile, docker-compose, fly.toml, k8s manifests, terraform

**Frontend signals**
- Meta-framework: next, remix, tanstack, nuxt, sveltekit, astro, gatsby, react-router v7
- UI lib: shadcn (usually a `components/ui/` folder), @radix-ui, @mui, @chakra-ui, @mantine
- Styling: tailwindcss, styled-components, @emotion, vanilla-extract, @pandacss
- State: @reduxjs/toolkit, zustand, jotai, pinia, vuex, mobx, @xstate
- Data: @tanstack/react-query, swr, @apollo/client, urql, rtk-query
- Forms: react-hook-form, formik, vee-validate + zod/valibot/yup
- i18n: next-intl, react-i18next, @formatjs, vue-i18n
- Stories: `.stories.tsx`/`.stories.ts`, `.storybook/` folder
- Flags: launchdarkly, growthbook, unleash, posthog, statsig
- Telemetry: @sentry/*, posthog-js, @datadog/browser-rum, segment, mixpanel
- Build: vite, webpack, turbopack, rspack, esbuild config files

**Testing / CI**
- Test runners: vitest, jest, playwright, cypress, pytest, go test
- CI: `.github/workflows/`, `.gitlab-ci.yml`, `.circleci/`, `buildkite/`

### 1c. Enumerate bounded contexts

If `features/`, `modules/`, `contexts/`, `packages/`, or `apps/` exists at a consistent level, list each subdirectory by name. Each becomes a candidate per-feature/module doc.

### 1d. Do not ask the user for anything derivable from the codebase.

---

## Phase 2: Propose Documents

Present a proposal shaped by the scan, **not** a fixed template. Structure:

```
Detected:
- <stack summary, one line>
- <project shape: monorepo / modular / layered / flat>
- <N bounded contexts: list>

Proposed docs for docs/agent/ (based on detected signals):

Always-on:
1. tech-stack.md — <why>
2. conventions.md — <why>
3. workflows.md — <why>

Signal-driven:
4. architecture.md — <signal that fired>
5. canonical-shape.md — <signal>
6. data-model.md — <signal>
...

Per-bounded-context (under docs/agent/features/):
- family.md, family-member.md, wealth.md, ...
  (hub: features/README.md)

Skip any? Add others? (reply with numbers to skip, or describe additions)
```

Wait for user response before proceeding. Record approved + user-added docs.

### Proposal rules

- Propose every fired signal; do not self-censor. User skips what they don't want.
- If bounded-context directories exist, default to per-context docs under `docs/agent/features/`. Name them after each directory.
- Cap: if >15 candidates, group under headers (Architecture / Backend / Frontend / Ops) for readability. Still propose all.
- If a catalog entry's signal is ambiguous, propose it with a `?` and note the uncertainty.

---

## Phase 3: Interview (grill-me style)

Interview the user **relentlessly** about every aspect of the doc plan until you reach shared understanding. Walk each branch of the decision tree; resolve dependencies between decisions one-by-one before moving on. Shallow is failure — every approved doc must leave this phase with resolved scope, opinion level, exclusions, and gaps.

### Rules

- **One question at a time.** Never batch.
- **Explore codebase first.** If the answer is in the code, read the code instead of asking. Do not waste a question on something the scan already reveals.
- **Every question ships with a recommendation AND a rationale.** Format:
  > **Recommended:** <concrete answer>
  > **Why:** <what problem this avoids, what signal supports it, or what past pattern the project already follows>
- **Follow the implication.** When the user answers, pick the next question *because* of that answer — do not jump to an unrelated baseline Q. Dependent decisions first.
- **Block on unknown.** If a required Q gets "I don't know", stop and explain exactly what downstream choice it blocks.
- **Stop condition.** Every approved doc has: (a) resolved scope, (b) opinion level, (c) exclusions, (d) per-doc gaps closed. Until then, keep grilling.

Track question count globally: **Q1:**, **Q2:**, etc. Never reset per topic.

### Branches to walk (in order, but skip any resolved by scan)

1. **Audience & tone** — who reads these docs; affects everything downstream
2. **Opinion level** — opinionated rules vs descriptive observations; affects every doc's voice
3. **Opinion-level exceptions** — docs that must stay prescriptive even if the rest is descriptive (e.g., canonical-shape, dependency-rules, security)
4. **Hidden conventions** — patterns not visible in linter/config
5. **Exclusions** — topics docs must NOT cover (secrets, unstable APIs, legal-sensitive flows)
6. **Per-doc scope** — for each approved doc, what's in vs out; anything non-obvious from code
7. **Per-bounded-context specifics** — for each `features/<name>.md`: key flow, historical context, deprecations, upcoming refactor
8. **Agent guardrails** — behaviors agents should follow when touching this codebase (e.g., "always run migrations after schema change") that don't live in any single existing doc

### Scan-gap prompts (examples, fire only when signal matched AND code alone won't produce a useful doc)

- **auth.md** — "Any provider-setup steps done outside code (dashboard config, env seeding)?"
- **cron-jobs.md** — "Guardrails when adding new jobs — idempotency? locking? timezone?"
- **feature-flags.md** — "Flags defined in dashboard only, in code only, or both?"
- **data-model.md** — "Any aggregate boundaries not obvious from folder layout?"
- **security.md** — "Authz rules that live in code vs business policy that isn't yet encoded?"
- **per-feature `<name>.md`** — "Anything about `<name>` not obvious from code — history, deprecations, upcoming rewrite?"

Do not proceed to Phase 4 until every approved doc has left the interview with its branches resolved. Follow [INTERVIEW.md](INTERVIEW.md) for interview flow.

---

## Phase 4: Generate Docs

Create `docs/agent/` in the project root. Use subfolders when warranted.

### Folder layout

Default flat. Switch to subfolders when any of the following:
- More than 6 approved docs
- Per-bounded-context docs approved (always goes under `features/`)
- Scan detected monorepo (may use `apps/` / `packages/` mirrors)

Example layouts:

**Flat** (small project)
```
docs/agent/
├── tech-stack.md
├── conventions.md
└── workflows.md
```

**Layered** (backend DDD)
```
docs/agent/
├── README.md
├── tech-stack.md
├── conventions.md
├── workflows.md
├── architecture.md
├── canonical-shape.md
├── dependency-rules.md
├── data-model.md
├── db-migrations.md
├── auth.md
├── http-layer.md
├── error-handling.md
├── cron-jobs.md
├── testing.md
├── deployment.md
└── features/
    ├── README.md
    ├── family.md
    ├── family-member.md
    └── wealth.md
```

**Frontend**
```
docs/agent/
├── README.md
├── tech-stack.md
├── conventions.md
├── workflows.md
├── framework.md
├── routing.md
├── ui-components.md
├── styling.md
├── state-management.md
├── data-fetching.md
├── forms.md
├── i18n.md
├── testing.md
└── build.md
```

**Monorepo**
```
docs/agent/
├── README.md
├── monorepo.md
├── tech-stack.md
├── workflows.md
├── apps/
│   ├── web.md
│   └── api.md
└── packages/
    ├── ui.md
    └── config.md
```

### Hub files

When using subfolders, create a hub `README.md` inside each subfolder with a table: doc name + one-line summary. Also create `docs/agent/README.md` at the top with links to every doc.

### Frontmatter (required in every file)

```markdown
---
title: Document Title
last_updated: YYYY-MM-DD
scope: [tech-stack | conventions | workflows | architecture | feature | testing | ...]
---
```

### Content guidelines

- **Describe capabilities, not file paths** — "auth via JWT middleware" not "see src/auth/jwt.ts". Exception: architecture.md and canonical-shape.md may show the canonical folder tree since it's the load-bearing structure.
- **Document concepts and domain terms** — stable across refactors
- **Concrete examples** — short snippets when helpful
- **Agent-optimized headers** — let agent skip to relevant section fast
- **No padding** — omit sections with nothing to say
- **Cross-reference** — `[conventions](../conventions.md)` instead of duplicating

---

## Phase 5: AGENTS.md

Create or update `AGENTS.md` in project root. Progressive disclosure — detail lives in linked docs, but the root file must still orient a cold agent in under 30 seconds.

### Template (new AGENTS.md)

```markdown
# <Project Name>

<2–3 sentence description. Sentence 1: what the project is + domain. Sentence 2: notable stack or architectural fact that shapes how agents should approach it. Sentence 3 (optional): key constraint (audience, scale, compliance, etc.).>

## Quick Reference

- **Package manager**: <pm>
- **Dev**: `<dev cmd>`
- **Build**: `<build cmd>`
- **Test**: `<test cmd>`
- **Lint**: `<lint cmd>`
- **Typecheck**: `<typecheck cmd>` *(only if non-standard)*

<Optional one-liner calling out a non-obvious command gotcha, e.g., "Run `bun run test` — not `bun test` — because the wrapper loads test env.">

## Agent Documentation

Structured docs for agents live under [docs/agent/](docs/agent/). Load the topic matching your task; the index lists every file with a one-line purpose.

- [Docs Index](docs/agent/README.md)
- [Tech Stack](docs/agent/tech-stack.md) — runtime, frameworks, deps
- [Conventions](docs/agent/conventions.md) — naming, patterns
- [Workflows](docs/agent/workflows.md) — dev/build/test commands
- [... other approved top-level docs, each with a short blurb ...]
- [Per-Feature Docs](docs/agent/features/) — per bounded context *(if applicable)*
```

### Rules

- **Description** = 2–3 sentences. Not one. Not a paragraph. Orient, don't summarize every feature.
- **Quick Reference** = labeled list, not a bare code block. Labels let agents scan fast.
- **Bridge line** before the docs list is required — explain *why* these links exist, not just list them.
- **Per-link blurb** — every docs/agent/ link gets one short `— purpose` suffix. Bare links force agents to open files to find out what's in them.
- Key commands = only non-obvious or non-standard ones.
- No inline rules or style guides — link to `docs/agent/` instead.
- No file-path documentation of source code (beyond the `docs/agent/` links themselves).
- No "always"/"never" directives inline — put those in linked docs.

### Existing AGENTS.md — decide: append vs rewrite

Read the existing file first, then pick a path:

**Append** (default — preserve user's work) when the existing AGENTS.md already has:
- a project description, AND
- commands or setup info

Action:
1. Identify what's already covered (description, commands, any existing doc links)
2. Append a `## Agent Documentation` section at the end using the docs list block from the template above — including the bridge line and per-link blurbs
3. Do **not** rewrite the existing description or commands — user owns those
4. In the completion message, call out: "Appended Agent Documentation section to existing AGENTS.md. Review for duplicate links if you had manual entries."

**Rewrite** (full template) when the existing file is a stub:
- Empty file, or
- Only a title, or
- Boilerplate with no real content (e.g., just `# AGENTS.md` placeholder)

Action: replace with the full template, preserving nothing.

**Never** silently overwrite description/commands the user has written. When in doubt, append.

---

## Completion

After all files are written, output:

```
Done. Created:
- docs/agent/[file list]
- AGENTS.md (updated/created)

Signals captured: [list]
Signals skipped: [list]

Next: use `update-agent-docs` skill after significant changes to keep docs current.
```
