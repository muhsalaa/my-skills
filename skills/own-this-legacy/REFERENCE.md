---
name: own-this
description: Reference — detailed scan, interview, and doc generation guide for the own-this skill.
---

# own-this — Reference

> Detailed companion to [SKILL.md](SKILL.md). Covers each stage in full depth
> with templates, techniques, and output formats.

---

## Stage 1 — Scan

Before saying anything to the user, silently read the project.

### What to look for

- **Entry points** — main files, index, app root, server start
- **Critical technologies** — frameworks, libraries, SDKs, external services
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

### Build internal checklist (by category)

Organise your findings into these categories. For each item, note its category
so the interview covers all dimensions — not just data flow.

| Category | What to look for |
|----------|------------------|
| **Tech stack** | Runtime (Node, Deno, Python, Go, Rust…), framework (Next, Express, Django, Gin…), language version, build tools (Vite, Turbopack, esbuild…), CSS approach (Tailwind, CSS Modules, vanilla…) |
| **Packages & dependencies** | Key dependencies from lockfile/package manifest — ORMs, DB drivers, auth libs, HTTP clients, validation libs, utility libs. Note which are central vs peripheral. |
| **Architecture & patterns** | Folder layout (layered, modular, flat), state management approach, data fetching pattern, component hierarchy, error handling strategy, middleware chain, dependency injection |
| **Data flow** | How data enters, transforms, and exits the system — API routes → handlers → services → DB, file upload pipelines, event pipelines |
| **Non-trivial implementations** | WASM, media processing, real-time (WebSockets, SSE), auth/encryption, file system ops, queues/workers, external API integrations |
| **Configuration & deployment** | Env vars shape, build/deploy configs, environment-specific behaviour, feature flags |
| **Risky spots** | Missing error handling, complex async chains, circular dependencies, mutable shared state, missing validation |

**Don't show this checklist to the user.** It's your guide, not a quiz sheet.

Prioritize by complexity and risk. Trivial things (a button click handler,
a simple utility) don't need coverage.

---

## Stage 2 — Interview

Start the conversation naturally. Introduce what you're doing in one or two
sentences, then dive in. Don't announce the structure or mention the checklist.

### Conversation principles

- **Conversational, not formal** — this is a discussion, not an exam.
- **Multiple-choice first** — always lead with a,b,c,d for each category.
- **Adaptive** — (a) → move on fast. (b) → 1-2 targeted follow-ups. (c) → note
  gap, move on. (d) → engage in discussion, they want to talk about it.
  Don't drag any category.
- **Natural topic transitions** — don't announce "moving to tech stack now". Let
  topics flow into each other organically. If a category hasn't come up naturally
  by the end, bring it up casually.
- **Accept honest answers** — if the user says "no idea" (c), that's gold.
  Don't pressure them. Move on.
- **Follow the user's energy** — if they're excited about a topic, let them talk.
  If they're clearly lost on something, note it and move on rather than making them
  feel bad.

### Category-specific questioning (multiple-choice + discuss option)

Lead with a **4-option familiarity question** for every category.
The 4th option (d) is an escape hatch — if the user picks it, they want to
discuss rather than just rate themselves. Engage conversationally.

```
Template for every category:

"How familiar are you with [topic]?
  (a) Understand it well
  (b) Used it but don't fully get it
  (c) First time / no idea
  (d) Let's discuss this"
```

**If (a):** Quick confirm ("cool, sounds good"), tag ✅, move to next category.
**If (b):** Ask 1–2 targeted follow-ups to pinpoint the gap, tag ⚠️.
**If (c):** Tag ❌, say "no worries, I'll cover it in the docs", move on.
**If (d):** They want to talk about it. Engage in open discussion — answer
questions, explain concepts, explore together. Tag based on what emerges.

---

**Example for each category:**

**Tech stack**
- "This project uses [Next.js / Express / Django / …]. How familiar are you with it?
  (a) Used it before, know it well  (b) Used it but gaps  (c) First time  (d) Let's discuss"
- (b) follow-up: "Any part of the setup that's confusing — routing, data fetching, middleware?"

**Packages & dependencies**
- "The project depends on [Prisma / Zod / Redis / …]. How well do you know it?
  (a) Know what it does  (b) Heard of it but not sure how it's used here
  (c) Never heard of it  (d) Tell me about it"
- (b) follow-up: "Want me to explain how it's wired in?"

**Architecture & patterns**
- "How's your understanding of the project structure?
  (a) I know where everything lives  (b) Rough idea but not the details
  (c) Lost  (d) Let's go through it together"
- (b) follow-up: "Any specific layer — routes, services, DB layer — that's fuzzy?"

**Data flow**
- "Do you have a mental model of how data moves through the app?
  (a) Yes, clear picture  (b) Some parts  (c) Not really  (d) Walk me through it"
- (b) follow-up: "Which part — API calls, DB queries, state management?"

**Non-trivial implementations**
- "The [WASM / queue / WebSocket / auth] part. How familiar?
  (a) Understand it well  (b) Used but don't fully get it
  (c) No idea  (d) Let's talk about it"
- (b) follow-up: "Want me to walk through the key parts quickly?"

**Configuration & deployment**
- "The [env vars / Dockerfile / deploy config]. Clear?
  (a) Yes  (b) Somewhat  (c) Haven't looked  (d) Explain it to me"
- (b) follow-up: "Anything specific you want explained?"

**Risky spots**
- "I spotted [specific pattern in code]. Were you aware of it?
  (a) Yes, intentional  (b) Saw it but not sure why
  (c) No, didn't notice  (d) Tell me more"
- If (b) or (c): Briefly explain the risk.

### Internal tracking

Track each topic internally as you go with these symbols:

| Symbol | Meaning |
|--------|---------|
| ✅ | Understood well — the user has a solid grasp |
| ⚠️ | Partial — knows the surface but not the depth |
| ❌ | Gap — doesn't understand this at all |
| ⏭ | Skipped — not covered, but needs a page regardless |

### Ending the interview

Wrap up warmly when all checklist items are covered (or consciously skipped):

> *"I think I have a good picture of where you're at with this. Let me put
> together the docs."*

---

## Stage 3 — Gap analysis (internal)

Before writing anything, consolidate your notes:

- **What did the user understand well?** — These get lighter treatment in docs.
- **What were the gaps?** — These get deeper treatment — more explanation,
  more context, failure points, debugging guides.
- **What critical things exist in the codebase that weren't covered?** —
  These get a page regardless of interview coverage.
- **What patterns or risks did the scan find that the user seemed unaware of?** —
  Flag these prominently.

This analysis determines which pages to create and how deep each one goes.

---

## Stage 4 — Doc generation

### Always create these two files

**`own-this/MOC.md`** — the central map listing and linking every page.

**`own-this/gaps.md`** — what the user didn't fully understand, with honest
framing and pointers on where to go deeper.

### Dynamically create topic pages

Create a page for each critical topic found in the scan, shaped by interview
results. There is no fixed list — pages emerge from the project and conversation.

Examples of pages that might exist (project-dependent):

| Page | What it covers |
|------|----------------|
| `wasm.md` | WASM integration, why it's used, how it's wired in |
| `ffmpeg-pipeline.md` | Processing chain, inputs, outputs, flags |
| `file-sdk.md` | What the SDK does, how the app uses it, key methods |
| `auth-flow.md` | Authentication end to end |
| `data-flow.md` | How data moves through the system |
| `architecture.md` | Overall structure and why it's shaped that way |
| `critical-paths.md` | Most important code paths for debugging |

**Page depth is proportional to the gap** — if the user understood WASM well,
the `wasm.md` page is a concise reference. If they had no idea how it worked,
`wasm.md` goes deep — explains the concept, explains the implementation,
explains what could go wrong.

### Page format (per topic)

Adapt this structure per topic:

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

### MOC format — `own-this/MOC.md`

```markdown
# <Project Name> — own-this

> Generated by the own-this skill on <date>.
> This is a knowledge base shaped by a comprehension interview, not
> auto-generated docs.
> Pages marked ⚠️ cover areas where understanding was partial.
> Pages marked ❌ cover areas that were gaps — read these carefully.

## Project overview
<2–3 sentence description of what the app does and its core technical shape.>

## Pages

| Page | What it covers | Understanding |
|------|----------------|---------------|
| [gaps](gaps.md) | What to learn next | — |
| [wasm](wasm.md) | WebAssembly integration | ❌ |
| [ffmpeg-pipeline](ffmpeg-pipeline.md) | Video processing chain | ⚠️ |
| [file-sdk](file-sdk.md) | File SDK usage | ✅ |

## Critical paths
<List the 2–3 most important flows to understand for debugging this app.>

## Riskiest spots
<What to watch. Where bugs are most likely to hide.>
```

### Gaps format — `own-this/gaps.md`

```markdown
# Gaps — what to learn next

These are areas where your understanding was thin or missing during the
interview. Prioritized by risk to the project.

## High priority
<Things that would block you from debugging or optimizing the core app.>

### <Topic>
- What you said: "<honest summary of what the user expressed>"
- What's actually happening: "<brief honest explanation>"
- What to study: <search terms, concept names>

## Medium priority
<Things that are important but won't block you immediately.>

## Low priority
<Nice to know, but the app will be fine without it.>
```

### Obsidian/wikilink support

At the end of doc generation, ask:

> *"Do you have an Obsidian vault? If yes, give me the path and I'll copy the
> docs there with wikilink formatting."*

If the user provides a path:

1. Copy all files to the vault path.
2. Convert all internal links to `[[wikilinks]]` format.
3. Ensure filenames follow Obsidian conventions (no special characters).
4. Update the MOC table to use wikilinks.

If the user declines, leave files in `./own-this/` with standard markdown links.

---

## Behavior rules (expanded)

- **Never make the user feel bad** about gaps — the whole point is they didn't
  understand it yet. Gaps are gold.
- **Be honest in the docs** — if something is genuinely complex, say so.
  Don't sugarcoat risky spots.
- **Write for the user, not for the code** — docs explain *why* and
  *what could go wrong*, not just *what*.
- **Don't document the obvious** — skip things any developer would know.
  Focus on what's specific to this stack or implementation.
- **Pages are living docs** — tell the user they can re-run `own-this` after
  learning more and the docs will be updated to reflect their improved
  understanding.
- **If `./own-this/` already exists**, warn before overwriting. Ask to merge,
  regenerate, or cancel. If merging, keep existing pages and add/update only
  the ones affected by new understanding.

---

## What this skill is NOT

- Not a code documentation generator (JSDoc, README generator, etc.)
- Not a code review tool
- Not a refactoring assistant
- Not a learning curriculum builder

It is specifically a **comprehension interview + gap-aware documentation** skill.
The interview is what makes it different from everything else.
