---
name: own-this
description: Guides a developer through a comprehension interview about their vibe-coded project, then generates modular, gap-aware documentation that helps them truly own the codebase. Use when the developer says "own-this", "own this project", "help me understand this codebase", "debrief this project", or asks for help understanding a project they built with AI assistance. Run from inside the project root.
---

# own-this

Take ownership of a vibe-coded project. Scan → interview → gap analysis → docs.
Output is a knowledge base shaped by the developer's actual understanding, not
auto-generated docs.

## Quick start

Run `own-this` from the project root:

```
# In your project directory:
agent> own-this
```

The agent will silently scan your project, then interview you conversationally
to surface understanding gaps, then generate docs in `./own-this/`.

## The 4 stages

### 1. Scan (silent)
Before talking to the user, read the project and build an internal checklist:
entry points, critical technologies, non-trivial implementations (WASM, media
pipelines, auth, real-time systems, complex async), data flow, risky spots,
configuration. Prioritize by complexity and risk. Never show the checklist
to the user.

See [REFERENCE.md](REFERENCE.md#stage-1--scan) for detailed scan guidelines.

### 2. Interview (conversational)
Interview the user about their understanding of each item on the checklist.
Conversational tone — not an exam. Mix open questions, concept checks, multiple
choice for familiarity, and honest probes ("if this broke at 2am, where would
you start?"). Track internally: ✅ understood / ⚠️ partial / ❌ gap / ⏭ skipped.

See [REFERENCE.md](REFERENCE.md#stage-2--interview) for interview techniques.

### 3. Gap analysis (internal)
Consolidate notes before writing. Topics understood well → lighter docs. Gaps
→ deeper treatment. Risks the user seemed unaware of → flagged.

### 4. Doc generation
Generate modular docs in `./own-this/`. Two guaranteed files:

- **`own-this/MOC.md`** — central map linking all pages, with understanding
  indicators (✅/⚠️/❌), critical paths, and riskiest spots.
- **`own-this/gaps.md`** — honest list of gaps prioritized by risk, with
  search terms to go deeper.

Additional topic pages emerge from the scan + interview (e.g., `wasm.md`,
`ffmpeg-pipeline.md`, `auth-flow.md`, `data-flow.md`). Depth is proportional
to the gap — concise reference if understood, deep treatment if not.

See [REFERENCE.md](REFERENCE.md#stage-4--doc-generation) for page formats
and MOC/gaps templates.

## Output location

- **Default:** `./own-this/` inside the project root.
- **Obsidian (optional):** After generation, ask:
  *"Do you have an Obsidian vault? If yes, give me the path and I'll copy the
  docs there with wikilink formatting."*
  If the user provides a path, copy all files there and convert internal links
  to `[[wikilinks]]`.

## Behavior rules

- **Never make the user feel bad** about gaps — gaps are the point.
- **Be honest in docs** — call out risky spots, don't sugarcoat.
- **Write for the user, not the code** — explain *why* and *what could go wrong*,
  not just *what*.
- **Don't document the obvious** — skip things any developer would know.
  Focus on what's specific to this stack or implementation.
- **Pages are living docs** — the user can re-run `own-this` after learning more,
  and the docs update to reflect improved understanding.
- **If `./own-this/` already exists**, warn the user before overwriting, and
  offer to merge or regenerate.

## See also

- [REFERENCE.md](REFERENCE.md) — detailed scan, interview, and doc generation guide
