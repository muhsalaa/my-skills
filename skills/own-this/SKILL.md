---
name: own-this
description: "Help a developer understand an unfamiliar codebase well enough to contribute fast. Trigger when user says own-this, own this, help me understand this repo, I want to contribute to this, walk me through this codebase, I inherited this code, or anything expressing they are new to a codebase and want to get up to speed. Topic-filtered mode: own-this auth, websockets scopes the session to those areas only."
---

# own-this

Get a developer from zero to confident contributor on an unfamiliar codebase.

**Flow:**

1. Scan the project silently
2. Interview the developer — one question at a time, relentlessly — until you genuinely understand what they know and don't know
3. Recap — show what you found (confident, gaps, trivial, not-asked). Let the developer correct or drill deeper before generating
4. Generate one `.md` per topic they need, shaped entirely by the interview

The docs exist to fill their gaps. If they know it, no file. If they don't, a file that teaches it — scoped to how this project uses it.

---

## Trigger

- `own-this` / `own this`
- `help me understand this codebase / repo`
- `I want to contribute to this`
- `walk me through this project`
- `I inherited this code`

**Topic mode:** `own-this <topic, topic>` — scopes everything to those topics only.

Run from the project root.

---

## Stage 1 — Scan (silent, before saying anything)

Read the project. Build an internal picture.

**Explore aggressively** — don't ask the developer things you can find yourself. Read the code, trace the imports, follow the data flow. If you can answer a question by looking at a file, look at the file.

Build your internal map:

- What does this project do? (one sentence)
- Entry point(s) — where does execution start?
- Language, runtime, platform
- Every non-trivial dependency — what it does, where it's used
- Non-trivial implementations: auth, real-time, file processing, API integrations, state management, build pipeline, background jobs, WASM, anything complex
- Key files — what each does, what it imports, what imports it
- Contributor paths — what will a new contributor most likely need to touch?
- Gotchas — things that aren't obvious but will bite someone new

Build an internal question tree. Each node is something the developer needs to understand to contribute. You'll walk this tree in the interview — one branch at a time, resolving each before moving on.

---

## Stage 2 — Interview

This is the core of the skill. Not a checklist. Not a form. A relentless one-question-at-a-time grilling — like a senior engineer stress-testing a new hire's understanding.

### Rules

**One question at a time.** Never list multiple questions. Ask one, wait for the answer, then ask the next.

**No opinions or recommendations.** Don't share your read of the code. Don't hint at the answer. Ask plainly and let the developer answer from their own understanding. Your job is to surface gaps, not fill them during the interview.

**Explore the codebase instead of asking when you can.** If a question can be answered by reading a file, read the file first. Only ask the developer about things the code can't tell you — their intentions, their understanding, their familiarity with a technology.

**Walk every branch.** Each topic has sub-questions. Don't move to the next topic until the current one is fully resolved. If the developer's answer reveals a gap, go deeper on that gap before moving on.

**Probe until you reach real understanding.** A vague answer ("yeah I think it uses JWT") is not resolved. Follow up: "Where exactly does token validation happen?" or "What triggers that?" Keep probing until the answer is specific and confident, or until the gap is clearly confirmed.

**Adapt to what you find.** If the developer clearly knows something well, move fast. If they're shaky, slow down and go deeper. The interview shape is driven by their actual understanding, not a fixed script.

### What to cover

Work through your internal question tree in this order. Don't skip any section.

**1. Project shape**
What does it do, who uses it, what's the core loop. One or two questions to confirm shared understanding.

**2. Dependencies — cover every non-trivial package, one at a time**
This section is mandatory and must not be skipped or rushed. For each non-trivial package found in the scan:

- Ask if they've used it before
- If yes: ask what it does in this project specifically — probe until the answer is concrete
- If no / shaky: note it as a doc gap, ask one follow-up to confirm ("so you're not sure what Drizzle is doing here?"), then move on — don't teach during the interview
- Cover every package. Don't group them. Don't skip ones that seem obvious.

**3. Architecture**
How does a request / event / CLI command flow through the system end to end. Probe each step — entry point, middleware, handlers, data layer, response.

**4. Key implementations**
Auth, real-time, file processing, external API integrations, background jobs — whatever's non-trivial in this codebase. One topic at a time, walk every branch.

**5. Contributor paths**
"If you needed to add X, where would you start?" Pick a realistic feature for this codebase. Probe their answer — which files, in what order, what would they need to understand first.

### Opening

Start with your one-sentence project summary and one question:

> "I've read through the repo. This looks like a [type] app that [does X], built on [core stack]. Does that match your understanding?"

Then go straight into the first real question.

### Closing

When the full question tree is walked and you have a clear picture of their gaps, close the interview and present the recap.

---

## Recap — between interview and docs

Before generating, show the developer what you found. This is their chance to correct misjudgments or request deeper coverage.

**Format:**

```
── Confident (no docs) ──
  ✓  <topic>                     <why you're skipping it>

── Gaps (docs queued) ──
  ◆  <topic>                     <why it's a gap>
  ◐  <topic>                     partial / shaky

── Trivial (skipped) ──
  1. <pkg>                       <why trivial>

── Not asked (skipped) ──
  5. <topic>                     <what wasn't covered>
```

- **✓** = they knew it. No doc.
- **◆** = clear gap. Doc queued.
- **◐** = partial understanding. Doc queued unless they say otherwise.
- Number trivial/not-asked so the developer can type a number to drill deeper.

**Follow with one question:**

> "Anything wrong above, or topics I should add? Type a topic name to drill deeper, or a number (e.g. 'ask 5') to interview on that. Otherwise I'll generate the docs. Obsidian? Drop your vault path."

If they correct a gap ("actually I know commander") → remove it from the doc queue. If they ask to drill on a number → go back to interview mode for that topic. Loop until they're satisfied, then generate.

---

## Stage 3 — Generate docs

One `.md` file per topic the developer needs. Driven entirely by what the interview revealed.

### What always gets generated

**`index.md`** — the entry point for the whole doc set. Not a map of maps. A useful starting document that orients the developer and links to everything else.

**`contributing.md`** — setup, how to run the project, common tasks, what to touch for typical contributions.

Everything else is gap-driven. Two types of gaps produce files:

**Package gaps** — any package the developer didn't know or was shaky on gets its own `.md`. One file per package. Named after the package (e.g. `drizzle.md`, `hono.md`, `zod.md`). This is non-negotiable — if a package was a gap in the interview, it gets a file.

**Implementation gaps** — any project-specific flow or pattern they couldn't explain clearly gets its own `.md` (e.g. `auth-flow.md`, `websocket-handling.md`).

### index.md format

```markdown
# <Project name>

## What this is

2–3 sentences. What the project does, who uses it, what kind of codebase it is.

## Start here

The fastest path to your first contribution:

1. [contributing.md] — get it running, understand the workflow
2. [file you're most likely to touch first based on their goals]
3. [topic file most relevant to their contribution area]

## How the project is structured

Directory tree with one-line descriptions. Reference real paths.

src/
routes/ — HTTP handlers, one file per resource
lib/db.ts — Drizzle client + all DB queries
middleware/ — auth, logging, error handling
...

## How a request flows through the app

Step by step, with file references. From entry point to response.
(Adapt for CLI: from invocation to output. For libraries: from import to usage.)

## Pages in this doc set

| File            | What it covers             |
| --------------- | -------------------------- |
| contributing.md | Setup + common tasks       |
| drizzle.md      | ORM used for all DB access |
| auth-flow.md    | How login and session work |
| ...             | ...                        |
```

### contributing.md format

```markdown
# Contributing

## Setup

Exact commands to get running locally. Env vars needed. Common setup errors.

## Running the project

Dev server, watch mode, build — exact commands.

## Running tests

How to run them. What they cover. How to add new ones.

## Common tasks

### Adding a [route / component / feature / command]

Which files to create or edit. In what order.

### Changing [X specific to this project]

Where the logic lives. What else gets affected.

## Things to know before you touch anything

Files or patterns with hidden complexity. Config that must match exactly. Global state.
```

### Topic file format (one per gap)

```markdown
# <Package name or topic>

## What this is

Plain language. No assumed knowledge. What problem does it solve?
Use an analogy if it helps. 2–4 sentences.

(Skip for project-specific topics like auth-flow — go straight to how it works here.)

## How this project uses it

The core section. What job does it do here specifically?

- Which files are involved — name them with real paths
- Step-by-step flow — what goes in, what happens, what comes out
- Reference actual code patterns, not abstract descriptions

## What to touch when contributing

"If you need to add X → start in `src/routes/auth.ts`"
"If you're changing how Y works → the logic lives in `lib/db.ts`"
Concrete and actionable.

## Gotchas

Things that aren't obvious and will bite someone new:

- Config that has to match exactly
- Async behavior that looks synchronous
- Side effects, global state, tricky ordering

## Go deeper

Search terms (not URLs) to understand this more fully.
```

---

## Obsidian output

If the user wants Obsidian:

- Write all files to their vault path under `own-this/<project-name>/`
- Use `[[wikilinks]]` for all internal links
- No special characters in filenames

---

## Rules

- **Explore before asking.** If the code can answer it, read the code. Only ask the developer about things the code can't tell you.
- **One question at a time.** Always. No lists.
- **Never hint at the answer.** Ask plainly. Surface gaps through what the developer can and can't explain.
- **Walk every branch.** Don't leave a topic half-resolved.
- **One file per topic.** Not a monolithic tech-stack file. Each gap gets its own `.md`.
- **Only write what they need.** Gaps get docs. Known things don't.
- **Always reference real files.** Every explanation names the actual file.
- **index.md and contributing.md always get generated.** Everything else is gap-driven.
- **No recap or summary files.** No "here's what you don't know" meta-doc. Just the useful files.
