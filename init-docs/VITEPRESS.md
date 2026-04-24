# VitePress — Local Reader

Self-contained install inside `docs/handbook/`. No project-root pollution. Local only — no deployment config.

## Pre-flight check

Before writing any VitePress files:

1. Run `node --version`. Parse major version.
2. Require Node ≥18. If missing or lower, abort this phase with the message:
   ```
   VitePress needs Node 18+. Detected: <what was found or "node not installed">.
   Docs are written and readable as plain markdown.
   Install Node ≥18 and re-run this skill's VitePress phase, or install VitePress manually inside docs/handbook/.
   ```
3. Record the outcome for the completion report.

## Package manager detection

Inspect project root for lock files, in this priority order:

| File | Package manager |
|------|-----------------|
| `pnpm-lock.yaml` | pnpm |
| `bun.lockb` / `bun.lock` | bun |
| `yarn.lock` | yarn |
| `package-lock.json` | npm |
| none found (non-Node project) | npm (fallback) |

Record `<detected-pm>` for install command and the user-facing run command.

## Files to write

### `docs/handbook/package.json`

```json
{
  "name": "handbook",
  "version": "0.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vitepress dev",
    "build": "vitepress build",
    "preview": "vitepress preview"
  },
  "devDependencies": {
    "vitepress": "^1.0.0"
  }
}
```

Pin `vitepress` to the latest stable 1.x. Do not include any theme or search plugin — built-in local search handles it.

### `docs/handbook/.vitepress/config.ts`

Generate with detected site title + fully resolved sidebar tree. Template:

```ts
import { defineConfig } from 'vitepress';

export default defineConfig({
  title: '<DETECTED_SITE_TITLE>',
  description: '<ONE_SENTENCE_DESCRIPTION_FROM_SCAN_OR_INTERVIEW>',
  cleanUrls: true,
  themeConfig: {
    search: {
      provider: 'local',
    },
    sidebar: [
      // Generated from approved doc set — see "Sidebar generation" below
    ],
    outline: [2, 3],
    socialLinks: [],
  },
});
```

No `nav:` key. No `footer:`. No `editLink:`. No `lastUpdated:` (the frontmatter field is for humans, not VitePress auto-display — VitePress's built-in lastUpdated reads git, which skews noisily during generation).

### Site title detection

Try in order, use first hit:

1. `package.json` `name` field (root) → titleize (`my-app` → `My App`)
2. `Cargo.toml` `[package].name`
3. `pyproject.toml` `[project].name` or `[tool.poetry].name`
4. `go.mod` module last segment
5. `composer.json` `name` last segment
6. Repo folder basename → titleize

Append ` · Handbook` so the browser tab reads `My App · Handbook`.

### Description source

Use the interview headline for `index.md` (the "overall project" opening sentence) if it was captured; otherwise fallback to a generated sentence from the scan (`<Stack> project handbook for new engineers.`).

## Sidebar generation

Build the sidebar programmatically from the approved doc set. Emit only groups that have items. Default order and grouping:

```ts
sidebar: [
  {
    text: 'Onboarding',
    items: [
      { text: 'Getting Started', link: '/getting-started' },
      { text: 'Tech Stack', link: '/tech-stack' },
    ],
  },
  {
    text: 'Contributing',
    items: [
      { text: 'How to Contribute', link: '/contributing' },
      { text: 'Conventions', link: '/conventions' },
      { text: 'Workflows', link: '/workflows' },
    ],
  },
  {
    text: 'Architecture',
    items: [
      // architecture.md, canonical-shape.md, dependency-rules.md,
      // data-model.md, http-layer.md, auth.md, error-handling.md, ...
      // (only approved ones)
    ],
  },
  {
    text: 'Operations',
    items: [
      // ci-cd.md, deployment.md, environments.md, testing.md,
      // observability.md, cron-jobs.md, background-jobs.md, ...
    ],
  },
  {
    text: 'Design Decisions',
    items: [
      // { text: 'Index', link: '/design-decisions/' },
      // one entry per NNNN-<slug>.md
    ],
  },
  {
    text: 'Features',
    items: [
      // { text: 'Index', link: '/features/' },
      // one entry per features/<name>.md
    ],
  },
  {
    text: 'Reference',
    items: [
      { text: 'Glossary', link: '/glossary' },
    ],
  },
  {
    text: 'For LLMs',
    items: [
      { text: 'Agent Docs Pointer', link: '/for-llms' },
    ],
  },
],
```

Rules:

- Emit a group only if it has ≥1 item after filtering to approved docs.
- Inside each group, order = the order the docs appear in the CATALOG grouping (Always-on / Architecture / Backend / Frontend / Ops). Alphabetize only within the ADR and Features groups.
- ADR group items read their title from the ADR frontmatter, not filename.
- Features group items read the directory name (titleized) for the text, link to `/features/<name>`.

## Install command

After writing the three files above, run from `docs/handbook/`:

| pm | command |
|----|---------|
| npm | `npm install` |
| pnpm | `pnpm install` |
| yarn | `yarn` |
| bun | `bun install` |

Do not add flags. Do not use `--silent`. If install fails, print the error and record the outcome; docs remain written.

## GitHub README

Write `docs/handbook/README.md` (not rendered by VitePress — GitHub-visible only):

```markdown
# Handbook

This folder holds the project's onboarding handbook. On GitHub it renders as plain markdown; locally it can be served by VitePress with sidebar navigation and full-text search.

## Read locally

Prerequisites: Node 18 or newer.

```bash
cd docs/handbook
<detected-pm> install       # only needed first time
<detected-pm> run dev
```

The dev server opens on http://localhost:5173 by default. Stop with Ctrl-C.

## Structure

- `index.md` — homepage
- `getting-started.md` — new engineer onboarding
- `contributing.md` — how to contribute
- `design-decisions/` — ADRs for major technical choices
- `features/` — per-context stories (if applicable)
- `for-llms.md` — pointer to machine-readable agent docs (if applicable)

See `.vitepress/config.ts` for the full sidebar.

## Editing

Every page has `last_updated` in its frontmatter — bump it when you edit. Sections marked `**TODO:**` or `Rationale: TBD` need a human to fill them in.
```

Use the literal `<detected-pm>` in the write — substitute the detected value at generation time.

## Run command for completion report

Print:

```
cd docs/handbook && <detected-pm> run dev
```

Substitute `<detected-pm>` with the resolved value.
