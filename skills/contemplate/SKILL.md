---
name: contemplate
description: Surface the why behind code — decisions, intent, constraints, and absences — from code-derived evidence, especially when ADRs/comments/docs are missing, thin, or untrusted. Infers reasoning from source, git history, naming, config, and metadata. Produces cited conclusions (evident) and hypotheses (plausible alternatives). Use when user says "contemplate", asks why code was shaped a certain way, or needs to reason about legacy/inherited/undocumented code.
---

# Contemplate

**Archaeological reasoning for code.** When there's no one left who knows the why,
read the traces the code left behind.

## Trigger

User says `"contemplate"`.

**Topic-filtered mode** — pass comma-separated topics to scope the pipeline:

- `contemplate auth flow` — only trace and infer around auth
- `contemplate caching, database layer` — multiple topics

Run from inside the project root directory.

---

## Pipeline

### Stage 1 — Intake

Start with an open prompt:

> "Tell me anything you know about this project — what it does, who built it, why it exists. It's OK if you don't know much."

If the user gives a rich answer, move on. If they say very little or "I don't know," follow up with 1–2 targeted questions:

- "What does this project do, even vaguely?"
- "Who built it or what team?"

It's fine if they know nothing. The intake is gravy — the code is the source.

### Stage 2 — Trace

Scan the codebase with a **why lens**, not a how lens. Start with an inventory, skip generated/vendor outputs, then read targeted paths. See [REFERENCE.md](REFERENCE.md) for full trace protocol.

Look for:

- **Decision points** — where a choice was made (library selection, architecture pattern, data structure)
- **Constraints** — shapes in the code that imply external pressure (env vars, config, platform limits)
- **Absences** — what's intentionally NOT here (no cache, no auth, no tests)
- **Module boundaries** — why files are grouped this way
- **Naming patterns** — names reveal intent

**Evidence sources**:

- Source code itself
- Git history — commit messages, blame, PR descriptions
- Package metadata — description, keywords, dependencies
- Config files — tsconfig, docker, CI, deployment
- `.env.example` / `.env.template`

**Hard exclude secrets:** never read `.env` or `.env.*`, except `.env.example` and `.env.template`.

### Stage 3 — Infer

For each trace, reason about the why. Two-tier output — see [REFERENCE.md](REFERENCE.md) for format rules.

- **Conclusions** — evident from the code. Strong inferences backed by direct citations (`file:line`, commit hash, or config path).
- **Hypotheses** — plausible alternatives. Multiple explanations where the why isn't clear, with supporting and undermining evidence.

### Stage 4 — Map

Connect inferences into cascading chains:

> Decision A forced constraint B, which explains absence C.

Decisions aren't isolated — they cascade. Map the dependency graph of reasoning.

### Stage 5 — Document

Generate output in `contemplate/` directory. See [REFERENCE.md](REFERENCE.md) for format rules.

If `contemplate/` already exists, do not overwrite silently. Ask whether to replace it or create a dated directory like `contemplate-YYYYMMDD/`.

- **`contemplate/README.md`** — always created. Entry point. Intro, how to read, markdown links to all other pages.
- **Dynamic pages** — emerge from what was found. No fixed list. Could be per-module decision pages, constraint traces, absences pages, cascade maps, or something else entirely. Content dictates.

Ask the user at the end:

> "Do you have an Obsidian vault? If yes, give me the path and I'll copy the docs there with wikilink formatting."

---

## Key rules

- **Independent from own-this** — different tool, different situation. No composition, no handoff.
- **Two-tier only** — conclusions and hypotheses. No confidence scores, percentages, or “high/low confidence” labels.
- **Cite every conclusion** — every 🔵 claim needs direct evidence (`file:line`, commit hash, or config path).
- **No mid-pipeline pauses** — intake is the only human touchpoint. After that, pure archaeology.
- **Topic mode scopes everything** — when topics are provided, only trace code paths related to those topics.
- **Secrets hard-excluded** — never read, reference, or infer from `.env` or `.env.*`, except `.env.example` and `.env.template`.
- **Write for the why** — every page explains reasoning, decisions, constraints. Not how the code works, but why it's shaped that way.
- **Format follows content** — some pages split into conclusions/hypotheses sections. Others use inline tags (🔵 / 🟡). Content dictates.