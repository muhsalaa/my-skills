---
name: add-agent-docs
description: Add a single new doc to an existing docs/agent/ folder. Asks the user which doc to add (or accepts one named in the request), runs a grill-me interview to resolve scope and gaps, generates the file with frontmatter, and appends links to the hub README and AGENTS.md. Refuses if docs/agent/ does not exist (run init-agent-docs first) or if the target filename already exists. Use when the user wants to add a new agent doc, extend existing agent documentation, says "add agent doc", "add a new doc to docs/agent", or names a single doc to author (auth, cron-jobs, data-model, etc.).
---

# Add Agent Docs

Adds **one** new doc to an existing `docs/agent/` folder. Reuses doc menu and content rules from init-agent-docs but stays incremental — no full rescan, no batch.

## Flow

1. **Check `docs/agent/` exists.** If missing, stop and tell the user to run `init-agent-docs` skill first. Do not bootstrap.

2. **Get the doc name.** If the user already named one in the request, use it. Otherwise ask: "What doc do you want to add?" (open question — do not show a menu). One doc per invocation — if the user names two, ask which goes first.

3. **Refuse on conflict.** If the target file already exists at the computed path (e.g., `docs/agent/auth.md` or `docs/agent/features/wealth.md`), stop. Tell the user to edit the existing file or pick a different name. Do not overwrite, do not merge.

4. **Interview (grill-me).** Resolve scope, opinion level, exclusions, and per-doc gaps before writing. One question at a time, every question with a recommendation + rationale. Read the code first when the answer lives there. See [INTERVIEW.md](INTERVIEW.md). Before interviewing, check [CATALOG.md](CATALOG.md) — if the doc name matches an entry, use its signal hints to scan the code first (saves interview turns) and its capture hints as the content checklist. If no match, skip the catalog and derive everything from interview + code exploration.

5. **Generate the file.** Write to the target path with required frontmatter:
   ```markdown
   ---
   title: <Document Title>
   last_updated: YYYY-MM-DD
   scope: [tech-stack | conventions | workflows | architecture | feature | testing | ...]
   ---
   ```
   Describe capabilities and concepts, not file paths. Cross-link other `docs/agent/` files instead of duplicating.

6. **Wire up links.** Append-only — never rewrite existing entries:
   - `docs/agent/README.md` (or the relevant subfolder hub like `features/README.md`) — add the new doc to the link table with a one-line purpose.
   - `AGENTS.md` — append a bullet under the existing `## Agent Documentation` section in the format `- [<Title>](docs/agent/<path>) — <one-line purpose>`. If the section is missing, append it. If `AGENTS.md` itself is missing, recommend running `init-agent-docs`.

## Constraints

- Single doc per invocation.
- Refuse on missing `docs/agent/` and on filename conflict — no exceptions.
- Same grill-me depth as init-agent-docs. Shallow interview is failure.
- Never edit existing docs other than hub README + AGENTS.md link list.
- `last_updated` uses today's date from system, not memory.

## Output

```
Done. Added:
- docs/agent/<path>
- Updated docs/agent/README.md (hub link)
- Updated AGENTS.md (docs list link)
```
