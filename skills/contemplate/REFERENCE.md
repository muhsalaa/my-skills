# Contemplate — Reference

---

## Trace protocol

### Full-trace mode (no topics)

Inventory the entire project, then read targeted source/config/history paths. For every trace, build an internal inference log (private — don't show the user).

**Skip generated/vendor paths unless directly relevant:** `.git/`, `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, `coverage/`, generated clients, lockfile-only noise, binaries, media, and minified bundles.

**Secrets rule:** never read `.env` or `.env.*`, except `.env.example` and `.env.template`.

**What to look for — the why lens:**

1. **Decision points**
   - Library/package choices — why this one and not the alternative?
   - Architecture patterns — why monolith vs microservices, REST vs GraphQL, etc.?
   - Data structure choices — why array vs map, why this schema shape?
   - API design — why this interface shape?
   - Error handling strategy — why try/catch here but not there?

2. **Constraints**
   - Config files reveal deployment reality — what environment, what platform, what limits?
   - `.env.example` / `.env.template` reveal which external services exist and what configuration the app needs
   - CI/CD config reveals deployment constraints and testing requirements
   - Docker/package config reveals runtime constraints
   - Import patterns reveal module dependency constraints

3. **Absences**
   - No caching layer — was consistency prioritized? Or wasn't it needed yet?
   - No auth — intentional (internal tool)? Or not built yet?
   - No tests — rushed? Or trivial codebase?
   - No error handling in a section — deliberate? Or oversight?
   - No logging — intentional (performance)? Or not implemented?

4. **Module boundaries**
   - Why are files grouped this way? What responsibility does each boundary serve?
   - Are boundaries clean (separation of concerns) or tangled (historical accident)?
   - Why is this code in a separate module vs inline?

5. **Naming patterns**
   - Function/variable names often encode intent: `safeParse`, `ensureAuthenticated`, `retryWithBackoff`
   - Directory names reveal architectural decisions: `adapters/`, `core/`, `legacy/`
   - Comment-less code still speaks through names

6. **Git evidence**
   - Commit messages — direct statements of intent
   - Blame — when was this added? In what context?
   - Branch names — feature branches reveal purpose
   - PR descriptions (if available) — the richest source of "why"

7. **Package metadata**
   - `description` and `keywords` in package.json — author's own summary
   - Dependency versions — pinned vs range reveals stability preferences
   - DevDependencies — tooling choices reveal workflow decisions

### Topic-filtered mode (topics provided)

Same trace structure, but **scoped to the topic list**. Do NOT trace the full project.

**How to trace a topic:**

1. Search for imports, references, filenames, and configuration related to the topic
2. Follow import chains: identify files that directly configure, import, or implement it
3. Read those files and note: what decisions were made, what constraints exist, what's absent
4. Map reasoning chains **through** that topic's code path only
5. Check git history for files related to the topic

**Build a topic-scoped inference log** — one entry per requested topic with the same categories (decisions, constraints, absences, naming, git evidence) but limited to the topic's trace path.

If a topic has no code trace, note that honestly in the output.

---

## Two-tier format rules

Every inference falls into exactly one tier:

### Conclusions 🔵

Evident from the code. Backed by direct evidence that would be hard to interpret differently. Every conclusion must include a citation: `path/to/file:line`, commit hash, or exact config path.

Examples of strong evidence for conclusions:
- A commit message says "feat: add retry for 429 rate limiting" — the retry exists because of rate limiting
- `.env.example` lists `REDIS_URL` — the app uses Redis (`.env.example`)
- Code uses `crypto.randomUUID()` instead of a UUID library — deliberate choice to use built-in (`src/id.ts:14`)
- A module is clearly named `auth-middleware` and every route passes through it — the app requires auth on all routes (`src/routes/index.ts:9`)

Mark as 🔵 in inline context, or use a "Conclusions" section header in structured pages.

### Hypotheses 🟡

Plausible alternatives. The code suggests something, but multiple explanations fit.

Examples of where hypotheses arise:
- No caching — could be intentional (consistency first), could be YAGNI, could be oversight
- A separate module exists — could be separation of concerns, could be planned reuse that never happened, could be legacy refactoring in progress
- A specific library was chosen — could be familiarity, could be feature fit, could be no better option existed at the time

When presenting hypotheses, **list alternatives** and explain what evidence supports and undermines each. Hypotheses do not need to be balanced; if undermining evidence is absent, say so explicitly:

```markdown
### Why is caching absent?

🟡 **Hypothesis 1: Consistency-first design** — The data source changes frequently
and stale reads would cause errors. Supporting evidence: real-time subscription
patterns elsewhere in the code (`src/events/subscribe.ts:12`). Undermining evidence:
no explicit comments or commits mention consistency.

🟡 **Hypothesis 2: Not needed yet** — The dataset is small enough that response
times are acceptable without caching. Supporting evidence: no performance-related
issues in git history, no optimization commits. Undermining evidence: there are
pagination/debouncing patterns elsewhere, so the author did think about performance.

🟡 **Hypothesis 3: Oversight** — Caching was never considered. Supporting evidence:
no cache-related dependencies, config keys, or TODOs exist. Undermining evidence:
other parts of the codebase show performance awareness (debouncing, pagination).
```

---

## Intake protocol

### Open prompt

Always start with:

> "Tell me anything you know about this project — what it does, who built it, why it exists. It's OK if you don't know much."

### Follow-up rules

- If the user gives a rich answer (2+ sentences with real context), move on. No follow-ups needed.
- If the user says very little or "I don't know," ask **one** targeted follow-up:

  > "What does this project do, even vaguely?"

- If they still don't know, ask:

  > "Who built it or what team?"

- If they still don't know, proceed without intake. The code is the source.

### Intake never makes the user feel required to know

The whole point of contemplate is that the why is missing. The intake is bonus context, not a prerequisite. Never pressure, never quiz, never show disappointment.

---

## Mapping — cascading decisions

Decisions chain together. Map these dependency graphs explicitly.

Example cascade:

```markdown
## Decision cascade: local-first architecture

🔵 **Decision: Local-first storage (SQLite)** (`src/db/client.ts:8`, `package.json`)
→ Constraint: Need to sync with server (`.env.example` contains `SYNC_URL`)
→ 🔵 **Decision: Bidirectional sync protocol** (`src/sync/push.ts:1`, `src/sync/pull.ts:1`)
→ Absence: No real-time collaboration — likely because CRDTs would be needed
  and weren't justified for single-user usage
→ 🟡 **Hypothesis: Offline-first was a product requirement, not just a tech choice**
```

These cascades are the most valuable output of contemplate — they connect isolated
inferences into a coherent story. Always look for cascading connections in Stage 4.

---

## Document generation

Before writing docs, check whether the target output directory exists. If it exists, ask whether to replace it or create `contemplate-YYYYMMDD/`.

### README.md — always created

```markdown
# <Project Name> — contemplate

> Generated by the contemplate skill on <date>.
> This is archaeological inference, not documentation. Conclusions are evident
> from the code. Hypotheses are plausible alternatives. Read critically.

## What this is

Reasoning traces for a codebase where the "why" is missing, thin, or untrusted.
ADRs/comments/docs may be absent, incomplete, or contradicted by code.

## How to read this

- 🔵 **Conclusions** — backed by direct evidence and citations.
- 🟡 **Hypotheses** — plausible alternatives. Could be any of them.
- Cascading chains show how one decision forced the next.

## Pages

| Page | What it covers | Key insight |
|------|----------------|-------------|
| [Auth decisions](auth-decisions.md) | Auth architecture reasoning | 🔵 Centralized middleware pattern |
| [Caching absence](caching-absence.md) | Why no cache exists | 🟡 Multiple hypotheses |
| ... | ... | ... |

## Key cascades

<List the most important decision chains found, 2-3 sentences each.>

## Open questions

<Things where even hypotheses are thin — the code gives almost no signal.
These are the places where you'd need to ask the original author.>
```

### Dynamic pages — format

No fixed structure. Let content dictate. Each page covers one reasoning cluster — a decision, a module, a pattern, an absence, a cascade.

Use the two-tier format within pages:

**Option A — structured sections:**

```markdown
# Why is auth centralized in middleware?

## Conclusions 🔵

- Every route passes through `auth-middleware.ts` — auth is required globally, not per-route (`src/routes/index.ts:9`)
- The middleware issues JWTs — implementing its own auth rather than delegating to a third party (`src/auth/auth-middleware.ts:42`)

## Hypotheses 🟡

**Why own auth instead of Auth0/Clerk?**
- 🟡 The app needs custom token claims beyond what managed auth provides (evidenced by custom payload in JWT)
- 🟡 The project started before managed auth was common — the git history shows auth was added early
- 🟡 Cost — managed auth services charge per user, and this app might have high user volume

## What this means for you

If you need to change auth, you're modifying the custom implementation.
If you add a route, auth is automatic — but you can't selectively opt out
without understanding the middleware chain.
```

**Option B — inline tags in narrative prose:**

```markdown
# Why this module structure

The codebase splits into three clear layers: `core/`, `adapters/`, and `handlers/`.
This is 🔵 a hexagonal architecture pattern — `core` has no external dependencies,
`adapters` implement interfaces for external services, and `handlers` wire everything together (`src/core/index.ts:1`, `src/adapters/db.ts:1`, `src/handlers/http.ts:1`).

The reason for this split is likely 🟡 driven by testability — core logic can be tested
without I/O by mocking adapters. Supporting evidence: test files only import from `core`
and never from `adapters` directly. Undermining evidence: no explicit architecture note
says testability was the goal. An alternative hypothesis is that this was modeled after
a similar project the author had worked on before.
```

Content dictates which format. Some pages naturally split into sections. Others flow better as narrative. Mix freely.

### Obsidian vault output

If the user provides an Obsidian vault path:

1. Copy all files to that path under a `contemplate/` folder
2. Convert internal markdown links like `[Auth decisions](auth-decisions.md)` to `[[auth-decisions|Auth decisions]]`
3. Ensure filenames match Obsidian conventions (lowercase slug, no special chars)

---

## Topic-filtered mode

When topics are provided, every stage scopes to those topics only:

- **Intake** — still open, but if the user mentions topics you can relate them
- **Trace** — only trace code paths related to the topics
- **Infer** — only produce inferences about those topics
- **Map** — only map cascades that involve those topics
- **Document** — only generate pages for those topics

The README still gets created but is scoped to the topic list. Dynamic pages are per-topic.

Parse the user message for topics after the `contemplate` trigger. Trim whitespace, split by commas. Example:

- `contemplate auth flow` → topics: ["auth flow"]
- `contemplate caching, database layer` → topics: ["caching", "database layer"]