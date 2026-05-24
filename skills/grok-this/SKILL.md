---
name: grok-this
description: "Help a developer understand an unfamiliar codebase fast enough to contribute. Trigger when user says grok-this, grok this, help me understand this repo, I want to contribute to this, walk me through this codebase, I inherited this code, or anything meaning I am new to this code and need to get up to speed. Works on any codebase type. Topic-filtered mode: grok-this auth, websockets scopes the whole session to those areas only."
---

# grok-this

Get a developer from zero to contributor on an unfamiliar codebase.

**How it works:**

1. Scan the project silently
2. Interview the developer to find out what they don't know
3. Generate one `.md` file per topic they need — each file teaches that thing from scratch, scoped to how it's used in this project

The docs are shaped by the interview. If you already know React, there's no `react.md`. If you don't, there is one — written for you specifically, not copied from the React docs.

---

## Trigger

- `grok-this` / `grok this`
- `help me understand this codebase / repo`
- `I want to contribute to this`
- `walk me through this project`
- `I inherited this code`

**Topic mode:** `grok-this <topic, topic>` — scopes everything to those topics only.

Run from the project root.

---

## Stage 1 — Scan (silent, before saying anything)

Read the project. Build an internal picture of:

**What is this project?**

- One-sentence description of what it does
- Entry point(s)
- Language, runtime, platform
- Frontend / backend / fullstack / CLI / library

**Every dependency** — read `package.json`, `pyproject.toml`, `go.mod`, etc. For each non-trivial package:

- What does it do?
- Where is it used in this project?
- Would a typical developer know it? (determines if it needs a 101 doc)

**Non-trivial implementations** — anything beyond basic CRUD:

- Auth, tokens, sessions
- Real-time (websockets, SSE)
- File/media processing
- External API integrations
- State management patterns
- Build/bundling pipeline
- Background jobs, queues
- WASM, native bindings

**Key files** — for each source file: what does it do, what does it import, what imports it.

**Contributor paths** — what files/areas will a new contributor most likely need to touch?

Build an internal checklist of topics to cover in the interview. Never show this to the user.

---

## Stage 2 — Interview

Short, conversational. Goal: find out what the developer doesn't know so the docs can fill exactly those gaps.

**Open with a one-line summary:**

> "OK — this is a [type] app that [does X], built with [core stack]. That right?"

**Then ask what they know.** Be concrete — name the actual packages:

> "I see this uses Hono, Drizzle ORM, and Zod. Which of those are you familiar with?"

**For each technology they're unfamiliar with:** make a note. That technology gets its own `.md` file.

**For implementations (auth flow, websockets, etc.):** ask how well they understand it in this project specifically. Not in general — here.

**Keep it short.** 5–10 questions max. You're not teaching in the interview — you're finding out what to write. The docs do the teaching.

**At the end ask:**

> "Do you want these docs formatted for Obsidian? If yes, drop your vault path."

---

## Stage 3 — Generate docs

One `.md` file per thing the developer needs. No fixed list — it comes from the interview.

### File naming

`grok-this/<topic>.md`

Examples of what gets generated (based on what the developer doesn't know):

- `hono.md` — if they haven't used Hono
- `drizzle.md` — if they're shaky on Drizzle ORM
- `zod.md` — if Zod was new to them
- `auth-flow.md` — if they don't understand how auth works in this project
- `websockets.md` — if real-time was unclear
- `project-structure.md` — always generated, it's the entry point
- `contributing.md` — always generated, setup + where to touch code

Things the developer already knows → no file. You're not documenting the whole project. You're filling their specific gaps.

---

### Every file follows this format:

```markdown
# <Topic>

## What this is

Plain language. No assumed knowledge. Explain it like the reader has never seen it.
Use an analogy if it helps. 2–4 sentences.

(Skip this section if it's a project-specific topic with no general concept to explain,
like "auth-flow" — in that case, go straight to how it works here.)

## How this project uses it

This is the core section. Explain:

- What job does this technology/pattern do in THIS project specifically
- Which files are involved — name them with paths
- Walk through the actual flow, step by step
- What data goes in, what comes out

Reference real code. Don't be abstract.

## What to touch when you contribute

"If you need to add X → start in `src/routes/auth.ts`"
"If you're changing how Y works → the logic lives in `lib/db.ts`"

Concrete, actionable. This is what makes a contributor fast.

## Gotchas

Things that aren't obvious and will bite you:

- Config that has to match exactly
- Async behavior that looks synchronous
- Side effects you wouldn't expect
- Common mistakes in this codebase

## Go deeper

Search terms (not URLs) if they want to understand this more fully.
```

---

### project-structure.md (always generate)

```markdown
# Project structure

## What this project does

One paragraph. Plain language.

## Entry point

`<path>` — what happens when the app starts / CLI runs / server boots.

## Directory layout

<directory tree with one-line descriptions>

src/
routes/ — HTTP route handlers
lib/ — shared utilities and DB client
middleware/ — auth, logging, error handling
...

## Key files

| File            | What it does                               |
| --------------- | ------------------------------------------ |
| `src/index.ts`  | App entry, registers middleware and routes |
| `src/lib/db.ts` | Drizzle client + DB connection             |
| ...             | ...                                        |

## How a request flows through the app

Step-by-step from HTTP request → response (or CLI invocation → output).
Reference actual files at each step.
```

---

### contributing.md (always generate)

```markdown
# Contributing

## Setup

Commands to get it running locally. Env vars needed. Common setup errors.

## Running the project

Dev server, watch mode, build — exact commands.

## Running tests

How to run them. What they cover. How to add new ones.

## Common tasks

### Adding a new route / endpoint / command

Which files to create or edit. In what order. Example.

### Adding a new database table / model

Which files to touch. Migration steps if applicable.

### Changing [core behavior specific to this project]

Where the logic lives. What else gets affected.

## Things to know before you touch anything

Files or patterns with hidden complexity. Why they're tricky.
```

---

## Obsidian output

If the user wants Obsidian format:

- Write all files to their vault path under `grok-this/`
- Use `[[wikilinks]]` for all internal references
- No special characters in filenames

---

## Rules

- **One file per topic.** Not one big tech-stack file. Each package, each concept, each implementation pattern gets its own `.md`.
- **Only write what they need.** If they know it, skip it. The docs are for their gaps, not for the whole project.
- **Always reference real files.** Every explanation names the actual file it lives in.
- **Teach in the docs, not in the interview.** The interview finds the gaps. The files do the teaching.
- **project-structure.md and contributing.md always get generated.** Everything else is gap-driven.
- **No recap files.** No `gaps.md`, no familiarity tables, no summary of what they don't know. Just the docs that fill the gaps.
