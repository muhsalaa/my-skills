---
name: add-docs
description: Add a single new doc to an existing docs/handbook/ folder. Asks the user which doc to add (or accepts one named in the request), runs a grill-me interview tailored to doc type (features story, ADR rationale, source-backed summary, glossary seed, or standard headline), generates the file with human-voice prose and frontmatter, wires the new entry into the VitePress sidebar when safe, and appends to the relevant hub README (features/design-decisions). Refuses if docs/handbook/ does not exist (run init-docs first) or if the target filename already exists. Use when user wants to add a handbook doc, extend handbook, says "add docs", "add handbook doc", "/add-docs", or names a single doc to author ("add auth.md to handbook", "add a feature doc for wealth").
---

# Add Docs

Adds **one** new doc to an existing `docs/handbook/`. Reuses the doc menu and interview rules from init-docs but stays incremental — no full rescan, no batch.

## Self-contained

This skill is standalone. It does not depend on init-docs, update-docs, or agent-side siblings. Signal + content catalog is duplicated in [CATALOG.md](CATALOG.md); keep in sync with init-docs/CATALOG.md when either is updated.

## Flow

1. **Check `docs/handbook/` exists.** If missing, stop. Tell the user to run `init-docs` first — do not bootstrap.

2. **Get the doc name.** If the user already named one in the request, use it. Otherwise ask one open question: "What doc do you want to add?" No menu, no catalog surfacing. One doc per invocation — if the user names two, ask which goes first.

3. **Refuse on conflict.** If the target file already exists at the computed path (e.g., `docs/handbook/auth.md`, `docs/handbook/features/wealth.md`, `docs/handbook/design-decisions/0007-use-drizzle.md`), stop. Tell the user to edit the existing file, use `update-docs` for a refresh, or pick a different name. Do not overwrite, do not merge.

4. **Resolve scope from doc-name shape.** Different doc kinds use different interview rules:

   | Target | Rule |
   |--------|------|
   | `features/<name>.md` | Story question: "Story for `<name>`: who uses it, why it exists, anything a new engineer must not break?" |
   | `design-decisions/NNNN-<slug>.md` | Rationale question: "Why `<chosen option>` over alternatives? One paragraph." Assign next `NNNN` automatically. Use ADR template from [CATALOG.md](CATALOG.md). |
   | `getting-started.md` | If root `README.md` has install/run sections → summarize + link, skip headline question. If thin/missing → interview. |
   | `contributing.md` | If root `CONTRIBUTING.md` (or `.github/CONTRIBUTING.md`, `docs/CONTRIBUTING.md`) exists → summarize + link-back, skip headline question. If missing → interview (branch naming, PR flow, review, commit style). |
   | `glossary.md` | Seed entries from bounded context names detected in scan. Ask one question for extras: "List 3–10 domain terms a new engineer won't know with one-line definitions." |
   | `for-llms.md` | Generate pointer page only if `docs/agent/` or `AGENTS.md` detected. Refuse otherwise. Skip headline question. |
   | Any other filename matching a [CATALOG.md](CATALOG.md) row | Scan for the catalog's signals to pre-fill factual content. 1 headline question: "One-paragraph headline for `<doc>`: what should the opening paragraph say to a new engineer encountering this for the first time?" |
   | Filename not in catalog | Open interview — ask what the doc covers. Still only 1 headline question at minimum. |

5. **FRESH `docs/agent/` seed bypass.** If a sibling `docs/agent/<same-filename>.md` exists, cross-check its factual claims against current scan (deps, tool names, folder patterns). If FRESH (no contradictions), rephrase its opening into handbook voice instead of asking the headline question. If STALE (contradictions), ignore it — proceed with the normal interview.

6. **Interview rules.**
   - One question at a time. Every question ships with a **Recommended** answer and **Why**.
   - Explore the codebase before asking. If the answer is in the code, read the code.
   - Block on "I don't know" when the answer gates the doc's opening paragraph.
   - User may skip — generated prose carries a visible `> **TODO:**` blockquote marker.

7. **Generate the file.** Write to the computed path with frontmatter:

   ```markdown
   ---
   title: <Human-readable title>
   last_updated: YYYY-MM-DD
   ---
   ```

   Content voice (mirrors init-docs/CATALOG.md): short paragraphs, why alongside what, concrete example, link out to external framework docs, cross-link handbook pages with relative paths, no source file paths unless the path is load-bearing architecture, no cross-links into `docs/agent/` (except `for-llms.md` which is itself the pointer page).

8. **Wire into VitePress sidebar** (detect + fallback).
   - Read `docs/handbook/.vitepress/config.ts`.
   - If the sidebar still matches init-docs' generated template signature (group names present: Onboarding, Contributing, Architecture, Operations, Design Decisions, Features, Reference, For LLMs — in expected order) → auto-insert the new entry into the correct group. Use catalog row's category to pick the group. Maintain group ordering rules from init-docs (ADR group by filename asc, Features by folder name, everything else by CATALOG group order).
   - If customized → print the exact TS snippet and name the target group. Do not edit. Example:
     ```
     Sidebar looks customized — not auto-editing. Paste this into your Architecture group:
         { text: 'Auth', link: '/auth' },
     ```
   - If `.vitepress/config.ts` is missing → skip this step silently (user declined VitePress install at init time).

9. **Wire hub READMEs.** Append only — never rewrite existing rows.
   - Adding `features/<name>.md` → append row to `docs/handbook/features/README.md` table: `| [<name>](./<name>.md) | <one-line story from interview> |`.
   - Adding `design-decisions/NNNN-*.md` → append row to `docs/handbook/design-decisions/README.md` table: `| [NNNN — <title>](./NNNN-<slug>.md) | <status> | <date> |`.
   - All other docs → no hub README to update.

10. **Never touch.** `docs/handbook/index.md`, root `README.md`, root `CONTRIBUTING.md`, any docs outside `docs/handbook/`.

## Constraints

- Single doc per invocation.
- Refuse on missing `docs/handbook/`. Refuse on filename conflict. No exceptions.
- Same grill-me depth as init-docs — shallow interview is failure.
- `last_updated` uses today's date from system, not memory.
- Human voice throughout. Bullet walls only appear inside factual subsections (dependency lists, command tables).

## Output

```
Done. Added:
- docs/handbook/<path>

Wired:
- .vitepress/config.ts sidebar (auto-inserted in <group>)        // or: print snippet message
- features/README.md (hub row)                                    // if applicable
- design-decisions/README.md (hub row)                            // if applicable

Open TODOs:
- <file>: <what's missing>                                        // if user skipped any questions

Next:
  cd docs/handbook && <detected-pm> run dev
```
