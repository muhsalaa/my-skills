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

### Build an internal checklist

Build a private map of topics that need to be covered in the interview.
**Don't show this to the user.** It's your guide, not a quiz sheet.

Prioritize by complexity and risk — trivial things (a button click handler,
a simple utility function) don't need coverage. Focus on things where
misunderstanding would cause real bugs or block optimization.

---

## Stage 2 — Interview

Start the conversation naturally. Introduce what you're doing in one or two
sentences, then dive in. Don't announce the structure or mention the checklist.

### Conversation principles

- **Conversational, not formal** — this is a discussion, not an exam.
- **Adaptive** — if the user explains something well, move on. If they're shaky,
  slow down, ask follow-ups, probe deeper.
- **Natural topic transitions** — don't announce "moving on to WASM now". Let
  topics flow into each other organically. If a topic hasn't come up naturally
  by the end, bring it up casually.
- **Mixed question formats:**
  - Open explanation: *"Walk me through what happens when a user uploads a file"*
  - Concept check: *"What does WASM actually do here? Explain it in your own words"*
  - Multiple choice for familiarity: *"How familiar are you with FFmpeg?
    (a) Used it before and understand it well (b) Used it but don't fully get it
    (c) First time encountering it"*
  - Honest probe: *"If this broke at 2am, where would you start looking?"*
- **Accept honest answers** — if the user says "I have no idea how this works",
  that's gold. Don't let them off with vague answers but don't pressure them either.
- **Follow the user's energy** — if they're excited about a topic, let them talk.
  If they're clearly lost on something, note it and move on rather than making them
  feel bad.

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
