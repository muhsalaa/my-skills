# Signal & Content Reference

**Not a menu.** The user picks any doc name they want — this file is *not* shown to them as a list of options. Use it only when the user-named doc happens to match an entry below; the matched row provides:

- **Signal** — what to grep / look for in the codebase before interviewing (saves the user from answering things the code reveals)
- **What to capture** — content checklist for the generated doc

If the user names a doc that is not in this file, skip the catalog entirely. Do not propose entries from here. Do not suggest "have you considered also adding X". User-driven only.

> Mirrors init-agent-docs/CATALOG.md content. Keep in sync if either is updated.

## Always-on (every project)

| Doc | Purpose | Source signal |
|-----|---------|---------------|
| tech-stack.md | Runtime, framework, key libs, path aliases | package.json / Cargo.toml / pyproject.toml / go.mod |
| conventions.md | Naming, formatting, patterns (beyond linter) | Biome/ESLint/Prettier + observed patterns |
| workflows.md | Dev, build, test, lint commands | scripts / Makefile / justfile |

## Architecture & structure (signal-driven)

| Doc | Signal to detect | What to capture |
|-----|------------------|-----------------|
| architecture.md | `domain/`, `application/`, `infrastructure/`, `core/`, `adapters/`, `ports/` — or multiple top-level layer folders | Layers, dependency direction, DI root |
| canonical-shape.md | Repeated folder pattern inside `features/`, `modules/`, `contexts/`, `packages/` | Flat vs nested, naming suffixes, container rule |
| dependency-rules.md | Multiple bounded contexts + cross-imports — or tooling like dependency-cruiser, nx constraints | Allowed / forbidden cross-module imports |
| monorepo.md | `apps/` + `packages/`, Turborepo/Nx/pnpm workspaces, `turbo.json`, `nx.json`, `pnpm-workspace.yaml` | Workspace layout, task graph, shared packages |
| features/<name>.md | Each directory under `features/`, `modules/`, `contexts/`, `apps/`, `packages/` | Per-context: responsibilities, key flows, caveats |

## Backend concerns (signal-driven)

| Doc | Signal | What to capture |
|-----|--------|-----------------|
| data-model.md | ORM + entities folder (Drizzle/Prisma/TypeORM/SQLAlchemy/Ecto) | Entities, shared kernel, aggregates |
| db-migrations.md | drizzle-kit, prisma migrate, alembic, goose, flyway | Migration flow, generate/apply commands |
| auth.md | better-auth, next-auth, auth0, clerk, passport, devise, django-allauth | Provider setup, session model, protected routes |
| http-layer.md | Hono / Express / Fastify / NestJS / Koa with custom middlewares | Middleware chain, validation, response helpers |
| error-handling.md | `exceptions/`, `errors/`, `error-codes/` folders, custom Exception base class | Exception hierarchy, error catalog, mapping to HTTP |
| cron-jobs.md | croner, node-cron, bullmq-repeat, celery-beat, sidekiq-cron, `cron/` folder | Scheduled jobs registry, add-a-job flow |
| background-jobs.md | bullmq, bee-queue, sidekiq, celery, rq, rabbitmq workers | Queues, workers, retry/backoff |
| adapters.md | `adapters/`, `integrations/` folder + external SDK deps (S3, SES, Stripe, Redis) | Concrete impls + interface contract |
| notifications.md | email/push/sms channels folder, zeptomail/sendgrid/ses + fcm/apns | Channel abstraction, delivery flow |
| security.md | `authorization.ts`, `policies/`, `abilities/`, row-level security code | Authz model, visibility rules |
| api-contracts.md | OpenAPI spec, tRPC routers, GraphQL schema, proto files | Contract source, code-gen flow |

## Frontend concerns (signal-driven)

| Doc | Signal | What to capture |
|-----|--------|-----------------|
| framework.md | next, remix, nuxt, sveltekit, astro, react-router v7 | Routing model, data loading, rendering mode |
| routing.md | `app/`, `pages/`, `routes/` tree, react-router config | Route layout, params, layouts/loaders |
| ui-components.md | shadcn components dir, Radix, MUI, Chakra, Mantine | Component library conventions, theming |
| styling.md | tailwindcss, css modules, styled-components, emotion, vanilla-extract, panda | Styling approach, tokens, theme |
| state-management.md | redux, zustand, jotai, pinia, vuex, mobx | Store shape, slice/atom conventions |
| data-fetching.md | @tanstack/query, swr, apollo, urql, rtk-query | Fetching conventions, cache keys, mutations |
| forms.md | react-hook-form, formik, vee-validate, felte + zod/valibot/yup | Form pattern, validation integration |
| i18n.md | next-intl, react-i18next, formatjs, vue-i18n, messages/ folder | Translation flow, locale detection |
| accessibility.md | eslint-plugin-jsx-a11y, axe deps, documented a11y rules | a11y conventions, audit tooling |
| stories.md | `.stories.tsx`, storybook config | Storybook setup, story patterns |
| feature-flags.md | launchdarkly, growthbook, unleash, posthog flags, statsig | Flag source, evaluation pattern |
| telemetry.md | sentry, posthog, datadog-rum, segment, mixpanel | Event taxonomy, error reporting |
| build.md | vite, webpack, turbopack, rspack, esbuild config files | Build flow, env handling, chunking rules |

## Shared / ops (signal-driven)

| Doc | Signal | What to capture |
|-----|--------|-----------------|
| testing.md | vitest, jest, playwright, cypress, pytest, go test + `tests/` folder | Test types, setup, run commands |
| deployment.md | Dockerfile, docker-compose, fly.toml, vercel.json, render.yaml, k8s manifests, terraform | Build image, envs, release flow |
| ci-cd.md | `.github/workflows/`, `.gitlab-ci.yml`, `buildkite/`, circleci config | Pipeline stages, required checks |
| environments.md | `.env.example`, `config/`, multiple env files | Env vars, config precedence |
| observability.md | pino/winston/zap + metrics lib (prom-client, otel) | Logging, metrics, tracing |
| scripts.md | `scripts/` folder with seed/reset/migration helpers | Each script's purpose + invocation |

## Lookup rules

- Match doc name → if hit, use **Signal** column to scan code before the interview, **What to capture** as content checklist.
- No match → ignore this file. Drive the doc fully from interview + targeted code reads.
- For per-feature docs (`features/<name>.md`), confirm the bounded-context name with the user.
- Scan only the signal(s) for the chosen doc. Never re-scan the whole codebase.
- Never surface this catalog to the user as a list of options.
