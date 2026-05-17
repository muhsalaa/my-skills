# Signal Reference (path mapping only)

**Not a menu.** Used by update-docs Phase 4 to map a doc filename to code paths/patterns for `git log`/`git diff` filtering. Lookup-only.

If the doc filename matches an entry below, use the **Signal** column as the search pattern (libraries, folder names, file extensions). Combine results with doc-body grep + per-feature folder match — never use this file alone.

If the doc filename is not in this file, skip the catalog and rely on doc-body grep + folder match only.

> Mirrors update-agent-docs/CATALOG.md content. Keep in sync if either is updated.

## Always-on

| Doc | Signal patterns to map |
|-----|------------------------|
| tech-stack.md | package.json / Cargo.toml / pyproject.toml / go.mod / lockfiles |
| conventions.md | biome.json / .eslintrc / .prettierrc / rustfmt.toml / ruff.toml / .editorconfig |
| workflows.md | package.json scripts, Makefile, justfile, Taskfile |
| getting-started.md | root `README.md` + package.json scripts |
| contributing.md | root `CONTRIBUTING.md` / `.github/CONTRIBUTING.md` / `docs/CONTRIBUTING.md` |
| glossary.md | `features/`, `modules/`, `contexts/`, `packages/` subfolder names |
| for-llms.md | `docs/agent/*.md`, `AGENTS.md` |

## Architecture & structure

| Doc | Signal patterns |
|-----|-----------------|
| architecture.md | `domain/`, `application/`, `infrastructure/`, `core/`, `adapters/`, `ports/` |
| canonical-shape.md | repeated subfolder pattern under `features/`, `modules/`, `contexts/`, `packages/` |
| dependency-rules.md | dependency-cruiser config, nx constraints, eslint import rules |
| monorepo.md | `turbo.json`, `nx.json`, `pnpm-workspace.yaml`, `lerna.json`, Cargo workspace |
| features/\<name>.md | `features/<name>/`, `modules/<name>/`, `contexts/<name>/`, `apps/<name>/`, `packages/<name>/` |

## Backend

| Doc | Signal patterns |
|-----|-----------------|
| data-model.md | drizzle-orm / prisma / typeorm / sqlalchemy / ecto + entities folder |
| db-migrations.md | drizzle-kit, prisma migrate, alembic, goose, flyway, migrations folder |
| auth.md | better-auth / next-auth / auth0 / clerk / passport / devise / django-allauth |
| http-layer.md | hono / express / fastify / nestjs / koa / axum / gin / fastapi / rails / django |
| error-handling.md | `exceptions/`, `errors/`, `error-codes/`, custom Exception class |
| cron-jobs.md | croner, node-cron, bullmq-repeat, celery-beat, sidekiq-cron, `cron/` |
| background-jobs.md | bullmq, bee-queue, sidekiq, celery, rq, rabbitmq workers |
| adapters.md | `adapters/`, `integrations/` + S3 / SES / Stripe / Redis SDK |
| notifications.md | email / push / sms folders, zeptomail / sendgrid / ses / fcm / apns |
| security.md | `authorization.ts`, `policies/`, `abilities/`, RLS code |
| api-contracts.md | OpenAPI spec, tRPC routers, GraphQL schema, proto files |

## Frontend

| Doc | Signal patterns |
|-----|-----------------|
| framework.md | next / remix / nuxt / sveltekit / astro / react-router config |
| routing.md | `app/`, `pages/`, `routes/` tree, react-router config |
| ui-components.md | `components/ui/` (shadcn), @radix-ui, @mui, @chakra-ui, @mantine |
| styling.md | tailwindcss, css modules, styled-components, emotion, vanilla-extract, panda |
| state-management.md | redux, zustand, jotai, pinia, vuex, mobx, xstate |
| data-fetching.md | @tanstack/query, swr, apollo, urql, rtk-query |
| forms.md | react-hook-form, formik, vee-validate + zod / valibot / yup |
| i18n.md | next-intl, react-i18next, formatjs, vue-i18n, `messages/` |
| accessibility.md | eslint-plugin-jsx-a11y, axe deps |
| stories.md | `*.stories.tsx`, `.storybook/` |
| feature-flags.md | launchdarkly, growthbook, unleash, posthog flags, statsig |
| telemetry.md | sentry, posthog, datadog-rum, segment, mixpanel |
| build.md | vite / webpack / turbopack / rspack / esbuild config |

## Shared / ops

| Doc | Signal patterns |
|-----|-----------------|
| testing.md | vitest, jest, playwright, cypress, pytest, go test + `tests/` |
| deployment.md | Dockerfile, docker-compose, fly.toml, vercel.json, render.yaml, k8s, terraform |
| ci-cd.md | `.github/workflows/`, `.gitlab-ci.yml`, `buildkite/`, circleci |
| environments.md | `.env.example`, `config/`, multiple env files |
| observability.md | pino, winston, zap + prom-client, otel |
| scripts.md | `scripts/` folder |

## Design decisions (ADR lifecycle only — no body edits)

| Slug pattern | Subject check |
|--------------|---------------|
| `NNNN-use-<tool>.md` | `<tool>` still in manifest deps? If absent → candidate for supersede |
| `NNNN-adopt-<pattern>.md` | folder/config still present? |
| `NNNN-migrate-to-<tool>.md` | `<tool>` in deps AND prior tool absent? |

ADR subject absent from code → flag only. Never rewrite ADR body. Offer status flip + scaffold new ADR.

## Lookup rules

- Match doc filename → use **Signal patterns** column for git diff path filter.
- Combine with doc-body grep (paths/symbols mentioned in the doc) and folder-name match.
- No catalog match → rely on doc-body grep + folder match only.
- Per-feature docs (`features/<name>.md`) → folder match dominates; catalog row is generic.
- ADRs use slug-pattern check, not path filter.
- Never surface catalog to user.
