---
name: own-this
description: Help developers take ownership of vibe-coded projects they built with AI but don't fully understand. Use this skill when a user says "own-this", "own this project", "help me understand this codebase", "I vibe-coded this and don't understand it", or "debrief this project". The skill scans the project, interviews the developer conversationally to surface genuine understanding gaps, then generates modular dynamic documentation shaped by the interview. Trigger whenever someone expresses shallow understanding of code they wrote with AI assistance, even if they don't use the exact phrase "own-this".
---

# own-this

**A skill for AI agents to help developers take ownership of vibe-coded projects.**

---

## Purpose

Developers who vibe-code with AI often end up with working apps they don't fully
understand. When bugs hit or optimization is needed, they're helpless without the AI.

This skill guides an agent to:

1. Scan a project and map its critical technologies and implementation points
2. Interview the developer conversationally to surface genuine understanding gaps
3. Generate modular, dynamic documentation based on what was found and what was unclear

The output is a knowledge base the developer actually owns — not generated docs, but
docs shaped by their own understanding.

---

## Trigger

User says any of:

- `"own-this"`
- `"own this project"`
- `"help me understand this codebase"`
- `"debrief this project"`

Run from inside the project root directory.

---

## Stage 1 — Scan

Before saying anything to the user, silently read the project.

**What to look for:**

- **Entry points** — main files, index, app root, server start
- **Critical technologies** — frameworks, libraries, SDKs, external services
- **Every dependency** — read `package.json` (or equivalent). For each dependency listed under `dependencies` and `devDependencies`, note what it does, why it might be used, and whether the developer is likely to know it. This becomes the tech-stack glossary.
- **Non-trivial implementations** — anything that isn't standard CRUD:
  - Binary/compiled integrations (WASM, native modules)
  - Media processing pipelines (FFmpeg, image manipulation)
  - Real-time systems (websockets, event streams)
  - Auth flows, encryption, token handling
  - Complex state management
  - File system operations, streams, buffers
  - External API integrations
  - Background jobs, queues, workers
- **Data flow** — how data enters, transforms, and exits the system
- **Risky spots** — error-prone patterns, missing error handling, complex async chains
- **Configuration** — env vars, build config, deployment-specific behavior

**Build an internal checklist** — a private map of topics that need to be covered
in the interview. Don't show this to the user. It's your guide, not a quiz sheet.

**Prioritize by complexity and risk** — trivial things (a button click handler,
a simple utility function) don't need coverage. Focus on things where misunderstanding
would cause real bugs or block optimization.

**Build a technology catalog** — for every non-trivial dependency, note:
- What is this package/library?
- What problem does it solve?
- What part of the project uses it? (which files)
- Would the developer likely know it? (verify in interview)

**Build a file map** — for every source file, note:
- What is this file's responsibility?
- What does it import?
- What imports it?
This becomes the code-map page.

---

## Stage 2 — Interview

Start the conversation naturally. Introduce what you're doing in one or two sentences,
then dive in. Don't announce the structure or mention the checklist.

**Conversation principles:**

- **Conversational, not formal** — this is a discussion, not an exam
- **Adaptive** — if the user explains something well, move on. If they're shaky, slow down, ask follow-ups, probe deeper
- **Natural topic transitions** — don't announce "moving on to WASM now". Let topics flow into each other organically. If a topic hasn't come up naturally by the end, bring it up casually
- **Accept honest answers** — if the user says "I have no idea how this works", that's gold. Don't let them off with vague answers but don't pressure them either
- **Explicitly ask about technology familiarity** — early in the interview ask about
their comfort with the core stack (language, runtime, major packages). Use a
multiple-choice / multi-select style question. For example:

  > "How familiar are you with the stack? I see [package A], [package B],
  > [package C] — which of those have you used before?"

  This is NOT about implementation — it's about whether they even know the tool.
  Gauge familiarity with: language/runtime, major dependencies (bundler, CLI libs,
  UI libs, test framework, etc.), build/publish pipeline, domain-specific tech.
  Track this dimension separately from implementation understanding (see below).
- **Follow the user's energy** — if they're excited about a topic let them talk. If they're clearly lost, note it and move on rather than making them feel bad

**Mixed question rhythm — keep it varied, never monotonous:**

The interview should feel like a conversation with a curious senior developer, not a quiz.
Deliberately alternate between question types so the user stays engaged and honest.

| Type                 | When to use                                                                | Example                                                                                                                                                         |
| -------------------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Open explanation** | Opening a new topic, testing depth                                         | _"Walk me through what happens when a user uploads a file"_                                                                                                     |
| **Multiple choice**  | Gauging familiarity quickly, resetting pace after a heavy topic            | _"How familiar are you with FFmpeg before this project? (a) Used it before and get it well (b) Used it but don't fully understand it (c) First time seeing it"_ |
| **Concept check**    | After they explain something — verify they know the why, not just the what | _"You mentioned WASM — explain in your own words what it actually does here"_                                                                                   |
| **Honest probe**     | Surfaces real understanding vs surface confidence                          | _"If this broke at 2am and you had no AI, where would you even start?"_                                                                                         |
| **Scenario**         | Tests applied understanding, not memorized answers                         | _"If a user uploads a 2GB file and the browser crashes — which part of the code is most likely responsible?"_                                                   |
| **Fill-in**          | Quick lightweight check, good after 2–3 heavy questions                    | _"The FFmpeg command runs inside \_\_\_ — what's your understanding of what that environment is?"_                                                              |

**Pacing rules:**

- Never ask two open-ended questions in a row — follow a heavy question with a lighter one
- Use multiple choice to reset after any topic where the user struggled — it relieves pressure and lets them recover
- Use scenarios sparingly — max 1–2 per interview, only for the most critical paths
- If the user gives a short or evasive answer, don't immediately follow with another question — reflect it back first: _"So you're saying you're not sure what happens after the encode step — is that right?"_ This gives them a chance to elaborate naturally
- If energy drops (short answers, "I don't know" repeatedly), switch to multiple choice for a few questions to rebuild momentum before going open again

**Track internally as you go — two dimensions:**

**Technology familiarity** (does the dev know the *tool/package* itself?):
- ✅ Knows this library/framework/tool well
- ⚠️ Heard of it, used a little, but not confident
- ❌ Never used it, or first exposure to this kind of thing

**Implementation understanding** (does the dev know how it's *used here*?):
- ✅ Understood well
- ⚠️ Partial — knows the surface but not the depth
- ❌ Gap — doesn't understand this at all
- ⏭ Skipped — not covered, needs a page anyway

**End the interview naturally** — when all checklist items are covered (or consciously
skipped), wrap up warmly. Something like: _"I think I have a good picture of where
you're at with this. Let me put together the docs."_

---

## Stage 3 — Gap analysis (internal)

Before writing anything, consolidate your notes:

- What did the user understand well? (These get lighter treatment in docs)
- What were the gaps? (These get deeper treatment — more explanation, more context)
- What critical things exist in the codebase that weren't covered? (These get a page regardless)
- What patterns or risks did you find in the scan that the user seemed unaware of?

This analysis determines which pages to create and how deep each one goes.

---

## Stage 4 — Doc generation

### Always create these five files:

**`own-this/MOC.md`** — the central map. Lists and links every page generated.
**`own-this/gaps.md`** — what the user didn't fully understand, with honest framing
and pointers on where to go deeper.
**`own-this/tech-stack.md`** — the technology catalog. Every non-trivial dependency
and tool in the project, with a short intro for each (what it is, why used here).
**`own-this/code-map.md`** — the file map. Every source file listed with its
responsibility, what it imports, and what imports it. Module dependency diagram.
**`own-this/architecture.md`** — system architecture. How the pieces fit together,
key data flows, CLI routing, the high-level shape.

### Dynamically create topic pages as needed:

Create a page for each critical implementation topic found in the scan, shaped by
interview results. There is no fixed list — the pages emerge from the project and
the conversation. Depth is proportional to the gap (see below).

Examples of pages that might exist (project-dependent):

- `wasm.md` — what WASM is, why it's used here, how it's wired in
- `ffmpeg-pipeline.md` — the processing chain, inputs, outputs, flags used
- `file-sdk.md` — what the SDK does, how the app uses it, key methods
- `auth-flow.md` — how authentication works end to end
- `data-flow.md` — how data moves through the system
- `install-pipeline.md` — a complex CLI command flow with rollback logic
- `critical-paths.md` — the most important code paths to understand for debugging

**Page depth is proportional to the gap** — if the user understood WASM well, the
`wasm.md` page is a concise reference. If they had no idea how it worked, `wasm.md`
goes deep — explains the concept, explains the implementation, explains what could go wrong.

### Base page format

All topic pages use this structure as a base — adapt as needed:

```markdown
# <Topic>

## What this is

One paragraph. Plain language. No assumed knowledge.

## Why this project uses it

Specific to this codebase — not generic documentation.

## How it's implemented here

Walk through the actual implementation. Reference real files and line areas.
Explain the non-obvious parts.

## What could go wrong

Common failure points, edge cases, things to watch.

## How to debug this

Where to look first if something breaks here.

## Go deeper

Concepts and search terms to study if you want to understand this fully.
(No URLs — search terms age better.)
```

### Special page: `tech-stack.md`

Intro to every non-trivial dependency and tool. Use this format:

```markdown
# Technology stack

## <Package name>

<!-- One entry per dependency/tool -->

### What this is

Plain language. What is this thing? What problem does it solve?

### Why this project uses it

What specific job does it do here?

### Where it's used

Key files that rely on this package.

### If you've never used it before

A quick concept primer — enough to follow the code.
```

Group entries into sections (e.g., "Runtime", "CLI", "Build", "Test").
For well-known universal concepts (Node.js `fs`, basic TypeScript), skip or
keep very brief. Focus on what someone new to the ecosystem wouldn't know.

### Special page: `code-map.md`

Every source file, its responsibility, and how they connect:

```markdown
# Code map

## Entry point

**`<path>`** — what this file does, how it's invoked

Imports from: <list>
Imported by: <list>

## Commands

**`<path>`** — what this command does, CLI args it handles

Imports from: <list>
Imported by: <list>

## Core modules

**`<path>`** — responsibility

Imports from: <list>
Imported by: <list>

## Utilities

**`<path>`** — responsibility

Imports from: <list>
Imported by: <list>

---

### Module dependency diagram

<ASCII or indented tree showing how modules depend on each other>
```

Each file gets one entry. Group by directory. The dependency diagram can be
an indented tree like:

```
cli.ts
 ├── commands/install.ts
 │    ├── core/config.ts
 │    ├── core/master.ts
 │    └── core/symlink.ts
 ├── commands/list.ts
 │    └── core/tracking.ts
 └── core/fs-utils.ts
```

### Wikilink format (Obsidian)

If the user wants output in an Obsidian vault, use `[[wikilinks]]` for all
internal page references instead of standard markdown links.

At the end of doc generation, ask:

> "Do you have an Obsidian vault? If yes, give me the path and I'll copy
> the docs there with wikilink formatting."

If they provide a path — copy all files there, convert all internal links to
`[[wikilinks]]`, and ensure filenames match Obsidian conventions (no special chars).

---

## MOC format — `own-this/MOC.md`

```markdown
# <Project Name> — own-this

> Generated by the own-this skill on <date>.
> This is a knowledge base shaped by a comprehension interview, not auto-generated docs.
> Pages marked ⚠️ cover areas where understanding was partial.
> Pages marked ❌ cover areas that were gaps — read these carefully.

## Project overview

<2–3 sentence description of what the app does and its core technical shape.>

## Pages

| Page                | What it covers                 | Understanding |
| ------------------- | ------------------------------ | ------------- |
| [[gaps]]            | What to learn next             | —             |
| [[tech-stack]]      | Technology & package glossary  | —             |
| [[code-map]]        | File layout & module deps      | —             |
| [[architecture]]    | System architecture & flow     | —             |
| [[wasm]]            | WebAssembly integration        | ❌            |
| [[ffmpeg-pipeline]] | Video processing chain         | ⚠️            |
| [[file-sdk]]        | File SDK usage                 | ✅            |

## Critical paths

<List the 2–3 most important flows to understand for debugging this app.>

## Riskiest spots

<What to watch. Where bugs are most likely to hide.>

## Technology familiarity

| Package | Familiarity | Doc page |
| ------- | ----------- | -------- |
| commander | ⚠️ Partial | [[tech-stack]] |
| @clack/prompts | ❌ New | [[tech-stack]] |
```

(The technology familiarity table tracks what the developer didn't know well,
so they can quickly see which tech-stack entries to read first.)
```

---

## Gaps format — `own-this/gaps.md`

```markdown
# Gaps — what to learn next

These are areas where your understanding was thin or missing during the interview.
Two categories: **technology familiarity** (tools/packages you haven't used before)
and **implementation understanding** (how things actually work in this codebase).

## Technology familiarity gaps

<Tools, libraries, or concepts the developer hasn't used before or is shaky on.
Pages in [[tech-stack]] cover these with plain-language intros.>

### <Tool/package name>

- Familiarity level: never used / heard of it / used a little
- What it is: <one-line explanation>
- Why this project needs it: <what job it does>
- Where to start: <which section of tech-stack.md to read first>

## Implementation understanding gaps

Prioritized by risk to the project.

### High priority

<Things that would block you from debugging or optimizing the core of the app.>

#### <Topic>

- What you said: "<honest summary of what the user expressed>"
- What's actually happening: "<brief honest explanation>"
- What to study: <search terms, concept names, which doc page covers this>

### Medium priority

<Things that are important but won't block you immediately.>

### Low priority

<Nice to know, but the app will be fine without it.>
```

---

## Behavior rules

- **Never make the user feel bad** about gaps — the whole point is they didn't understand it yet
- **Be honest in the docs** — if something is genuinely complex, say so. Don't sugarcoat risky spots
- **Write for the user, not for the code** — docs explain _why_ and _what could go wrong_, not just _what_
- **Don't document the obvious** — skip things any developer would know. Focus on what's specific to this stack or implementation
- **Pages are living docs** — tell the user they can re-run `own-this` after learning more and the docs will be updated to reflect their improved understanding
- **Introduce unfamiliar technology** — if a package or tool is new to the user,
  its doc page should include a plain-language "what is this" section. Don't assume
  they know what `@clack/prompts` or `commander` or `rolldown` is. A few sentences
  explaining the concept goes a long way.
