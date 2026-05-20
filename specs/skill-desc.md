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

---

## Stage 2 — Interview

Start the conversation naturally. Introduce what you're doing in one or two sentences,
then dive in. Don't announce the structure or mention the checklist.

**Conversation principles:**

- **Conversational, not formal** — this is a discussion, not an exam
- **Adaptive** — if the user explains something well, move on. If they're shaky, slow down, ask follow-ups, probe deeper
- **Natural topic transitions** — don't announce "moving on to WASM now". Let topics flow into each other organically. If a topic hasn't come up naturally by the end, bring it up casually
- **Mixed question formats:**
  - Open explanation: *"Walk me through what happens when a user uploads a file"*
  - Concept check: *"What does WASM actually do here? Explain it in your own words"*
  - Multiple choice for familiarity: *"How familiar are you with FFmpeg? (a) Used it before and understand it well (b) Used it but don't fully get it (c) First time encountering it"*
  - Honest probe: *"If this broke at 2am, where would you start looking?"*
- **Accept honest answers** — if the user says "I have no idea how this works", that's gold. Don't let them off with vague answers but don't pressure them either
- **Follow the user's energy** — if they're excited about a topic let them talk. If they're clearly lost on something, note it and move on rather than making them feel bad

**Track internally as you go:**
- ✅ Understood well
- ⚠️ Partial — knows the surface but not the depth
- ❌ Gap — doesn't understand this at all
- ⏭ Skipped — not covered, needs a page anyway

**End the interview naturally** — when all checklist items are covered (or consciously
skipped), wrap up warmly. Something like: *"I think I have a good picture of where
you're at with this. Let me put together the docs."*

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

### Always create these two files:

**`own-this/MOC.md`** — the central map. Lists and links every page generated.
**`own-this/gaps.md`** — what the user didn't fully understand, with honest framing
and pointers on where to go deeper (search terms, concepts to study, not specific URLs).

### Dynamically create topic pages as needed:

Create a page for each critical topic found in the scan, shaped by interview results.
There is no fixed list — the pages emerge from the project and the conversation.

Examples of pages that might exist (project-dependent):
- `wasm.md` — what WASM is, why it's used here, how it's wired in
- `ffmpeg-pipeline.md` — the processing chain, inputs, outputs, flags used
- `file-sdk.md` — what the SDK does, how the app uses it, key methods
- `auth-flow.md` — how authentication works end to end
- `data-flow.md` — how data moves through the system
- `architecture.md` — the overall structure and why it's shaped that way
- `critical-paths.md` — the most important code paths to understand for debugging

**Page depth is proportional to the gap** — if the user understood WASM well, the
`wasm.md` page is a concise reference. If they had no idea how it worked, `wasm.md`
goes deep — explains the concept, explains the implementation, explains what could go wrong.

### Page format

All pages use this structure as a base — adapt as needed per topic:

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

| Page | What it covers | Understanding |
|------|---------------|---------------|
| [[gaps]] | What to learn next | — |
| [[wasm]] | WebAssembly integration | ❌ |
| [[ffmpeg-pipeline]] | Video processing chain | ⚠️ |
| [[file-sdk]] | File SDK usage | ✅ |

## Critical paths
<List the 2–3 most important flows to understand for debugging this app.>

## Riskiest spots
<What to watch. Where bugs are most likely to hide.>
```

---

## Gaps format — `own-this/gaps.md`

```markdown
# Gaps — what to learn next

These are areas where your understanding was thin or missing during the interview.
Prioritized by risk to the project.

## High priority
<Things that would block you from debugging or optimizing the core of the app.>

### <Topic>
- What you said: "<honest summary of what the user expressed>"
- What's actually happening: "<brief honest explanation>"
- What to study: <search terms, concept names>

## Medium priority
<Things that are important but won't block you immediately.>

## Low priority
<Nice to know, but the app will be fine without it.>
```

---

## Behavior rules

- **Never make the user feel bad** about gaps — the whole point is they didn't understand it yet
- **Be honest in the docs** — if something is genuinely complex, say so. Don't sugarcoat risky spots
- **Write for the user, not for the code** — docs explain *why* and *what could go wrong*, not just *what*
- **Don't document the obvious** — skip things any developer would know. Focus on what's specific to this stack or implementation
- **Pages are living docs** — tell the user they can re-run `own-this` after learning more and the docs will be updated to reflect their improved understanding
