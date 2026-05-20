# own-this — Discussion Summary

_For use as reference when comparing or commissioning alternative skill implementations._

---

## The problem being solved

Developers who vibe-code with AI end up with working apps they don't fully understand.
The danger is twofold:

- When bugs hit, debugging is nearly impossible without AI assistance
- Optimization decisions are made blindly without understanding the underlying tech

The goal is not to read generated documentation — it's to build a genuine working
mental model of the project. The difference matters: passive reading doesn't create
the same understanding as being asked to explain something.

---

## The skill name

**`own-this`** — chosen deliberately for its ownership framing. Not "understand this"
or "document this" but _own_ it. The name carries the intention: take responsibility
for the code, not just familiarity with it.

Trigger phrases: `"own-this"`, `"own this project"`, `"help me understand this codebase"`

---

## The four stages

### 1. Scan (silent, before talking to user)

Agent reads the project cold and builds an internal checklist of critical topics.
Prioritizes by complexity and risk — trivial code is ignored.

Looks for: entry points, critical technologies, non-trivial implementations
(WASM, FFmpeg, auth flows, real-time systems, complex async), data flow,
risky spots, configuration.

The checklist is **never shown to the user** — it's the agent's guide, not a quiz sheet.

### 2. Interview (conversational)

The core of the skill. Agent interviews the user using a mix of:

- Open explanation questions ("walk me through what happens when...")
- Concept checks ("explain WASM in your own words")
- Multiple choice for familiarity level
- Honest probes ("if this broke at 2am, where would you start?")

**Key design decisions:**

- **Conversational, not formal** — no exam feel, no announced topic transitions
- **Adaptive** — drills deeper where user is shaky, moves on where they're solid
- **Natural transitions** — agent tracks checklist invisibly, steers organically
- **Honest gaps are gold** — "I have no idea" is a perfect answer, not a failure

Agent tracks each topic internally as: ✅ understood / ⚠️ partial / ❌ gap / ⏭ skipped

### 3. Gap analysis (internal, after interview)

Agent consolidates notes before writing anything:

- What was understood well → lighter treatment in docs
- What were the gaps → deeper treatment, more explanation
- What exists in codebase but wasn't covered → gets a page regardless
- What risks did the scan find that the user seemed unaware of

### 4. Doc generation (dynamic, modular)

Output is a knowledge base shaped by the interview — not a template, not auto-generated docs.

---

## Output structure philosophy

**Dynamic, not predefined.** Pages emerge from the project and conversation.
No fixed list of pages beyond two guaranteed files.

**Always exists:**

- `own-this/MOC.md` — central map, links all pages, marks understanding level per topic
- `own-this/gaps.md` — honest gap list, prioritized by risk, with search terms to go deeper

**Everything else is dynamic** — created based on what the scan found and what
the interview revealed. A file management app with WASM + FFmpeg might produce:
`wasm.md`, `ffmpeg-pipeline.md`, `file-sdk.md`, `upload-flow.md`.
A REST API might produce: `auth-flow.md`, `database-schema.md`, `rate-limiting.md`.

**Page depth is proportional to the gap** — topics the user understood well get
concise reference pages. Topics that were gaps get deep treatment: concept explanation,
implementation walkthrough, failure points, debugging guide.

---

## Output location

- **Default:** `./own-this/` inside the project root
- **Obsidian (optional):** At the end of generation, agent asks:
  > "Do you have an Obsidian vault? If yes, give me the path and I'll copy the docs there with wikilink formatting."
  - If user provides path → copy files there, convert all internal links to `[[wikilinks]]`
  - No assumption made — purely optional offer

Output is **standalone** — not integrated with any other system (e.g. learning wiki).
The user can manually link from their Obsidian notes if they choose.

---

## MOC design

Central page includes:

- Project overview (2–3 sentences)
- Table of all pages with understanding indicator (✅ / ⚠️ / ❌)
- Critical paths — 2–3 most important flows for debugging
- Riskiest spots — where bugs are most likely to hide

---

## Page format (per topic)

Each dynamic page follows this base structure (adapted per topic):

1. What this is — plain language, no assumed knowledge
2. Why this project uses it — specific to this codebase
3. How it's implemented here — references real files, explains non-obvious parts
4. What could go wrong — failure points, edge cases
5. How to debug this — where to look first
6. Go deeper — search terms and concept names (no URLs, they age poorly)

---

## Tone and behavior rules

- Never make the user feel bad about gaps — gaps are the point
- Be honest in docs — risky spots are called out, not sugarcoated
- Write for the user, not for the code — why and what could go wrong, not just what
- Don't document the obvious — focus on what's specific to this stack
- Pages are living docs — user can re-run `own-this` after learning more

---

## Connection to broader learning system

This skill was designed alongside a **learning wiki system** (based on Karpathy's
LLM Wiki pattern) that the same user uses to compile knowledge from YouTube, articles,
podcasts, and PDFs into a persistent Obsidian-based knowledge base.

`own-this` is intentionally **standalone and separate** from that system. However,
the `gaps.md` output could naturally be used as a source to feed into the learning
wiki — the user manually decides if they want to bridge the two systems.

---

## What this skill is NOT

- Not a code documentation generator (JSDoc, README generator, etc.)
- Not a code review tool
- Not a refactoring assistant
- Not a learning curriculum builder

It is specifically a **comprehension interview + gap-aware documentation** skill.
The interview is what makes it different from everything else.
