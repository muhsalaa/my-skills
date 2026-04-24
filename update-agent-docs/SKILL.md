---
name: update-agent-docs
description: Refresh existing docs/agent/ files against code changes since each doc's last_updated date. Two modes — auto (scan every doc, default when no args) and manual (user names specific docs). Both modes use git log committed-only diff against the doc's last_updated date, map docs to code paths via doc-body grep + catalog signals + per-feature folder match, perform surgical edits, then bump last_updated to today. Reports up-to-date docs and bumps their timestamp anyway to shrink future diff windows. Flags AGENTS.md drift instead of auto-syncing. Use when user wants to refresh agent docs, sync docs to recent code changes, says "update agent docs", "refresh docs/agent", or names docs to refresh ("update auth.md").
---

# Update Agent Docs

Refreshes `docs/agent/*.md` against code changes since each file's `last_updated` frontmatter date. Surgical edits, committed-only diff.

## Flow

1. **Check `docs/agent/` exists.** If missing, stop and tell the user to run `init-agent-docs` first.

2. **Pick the doc set.**
   - **No args** (default = auto): list every `.md` under `docs/agent/` (excluding `README.md` hubs) with its `last_updated`. Then confirm:
     ```
     Found N docs. Scan all (auto) or pick docs to update (manual)?
     ```
   - **User named docs** (manual): use only those. Validate each exists; refuse on missing path.

3. **Read frontmatter per doc.** If `last_updated` is missing, ask the user what date to diff from (today / specific date / first commit of file). Do not guess.

4. **Map doc → code paths** (combined, fall back gracefully):
   - **(a) Doc-body grep** — extract any file paths, folder names, symbols, or library names mentioned in the doc body.
   - **(b) Catalog signals** — if doc filename matches an entry in [CATALOG.md](CATALOG.md), pull its signal patterns (e.g., `auth.md` → search for better-auth / next-auth / clerk usages).
   - **(c) Per-feature folder match** — `features/<name>.md` maps to `features/<name>/`, `modules/<name>/`, `apps/<name>/`, or `packages/<name>/` if any exist.
   - Union the results into the path filter for git diff.

5. **Diff committed changes.** Run:
   ```
   git log --since=<last_updated> --name-only --pretty=format: -- <mapped paths>
   ```
   Then `git diff <last_updated_commit_or_date>..HEAD -- <changed files>` to see actual content. **Committed only** — ignore working tree.

6. **Upfront summary.** Print one line per doc:
   ```
   - auth.md (last 2026-02-14): 7 commits, 12 files changed
   - cron-jobs.md (last 2026-03-01): 0 commits — up to date
   - data-model.md (last 2026-01-09): 3 commits, 4 files changed
   ```
   Then proceed per doc.

7. **Per-doc handling**:
   - **No relevant changes** → report `up to date`, bump `last_updated` to today, move on. Bumping shortens the next run's diff window.
   - **Has changes** → confirm `update auth.md? (y/n/skip)`. On yes: surgical edit only the affected sections. Do not rewrite untouched sections. Do not change frontmatter `title` / `scope`.
   - After successful edit, bump `last_updated` to today.

8. **Detect undocumented areas.** Scan the full diff window (all commits since earliest `last_updated` in the set), not just paths mapped to existing docs. Flag signals that have **no matching agent doc**:
   - New manifest deps with catalog-known purpose (e.g., `stripe`, `bullmq`, `sentry`) and no doc covers it.
   - New top-level feature folders (`features/<new>/`, `modules/<new>/`, `apps/<new>/`, `packages/<new>/`) with no `features/<new>.md`.
   - New cross-cutting folders (`cron/`, `queue/`, `webhooks/`, `adapters/`, `migrations/`) with no matching doc.
   - New infra surface (first `Dockerfile`, first `.github/workflows/*`, first `terraform/`) with no `deployment.md` / `ci-cd.md`.

   If any found, print after per-doc work:
   ```
   Possibly undocumented (found in diff, no matching doc):
   - stripe dep added → suggests billing.md
   - features/reports/ created → suggests features/reports.md
   - .github/workflows/ added → suggests ci-cd.md

   Create new docs for these? (y/n)
   ```
   On yes: tell user to run `add-agent-docs` with the suggested slugs. **Do not create docs here** — `add-agent-docs` owns additions. No auto-invoke.

9. **Flag AGENTS.md drift.** If any updated doc was `workflows.md`, `tech-stack.md`, or any doc whose content tends to surface in AGENTS.md Quick Reference (commands, package manager), print a flag:
   ```
   ⚠ workflows.md changed. AGENTS.md Quick Reference may be stale — review manually.
   ```
   Do not edit AGENTS.md.

## Constraints

- Committed only. Working tree dirty? Ignore. Tell user to commit first if they want those changes considered.
- Surgical edits only. Section untouched by diff stays untouched.
- Always bump `last_updated` after a doc is processed (whether changed or up-to-date) — shrinks future diff windows.
- Per-doc confirm before editing in auto mode. Bulk-yes is the user's call.
- Never edit AGENTS.md. Flag drift only.
- Never delete docs. Removed code → leave the doc; user decides whether to delete.
- Never create new docs. Undocumented areas surfaced as suggestions only — hand off to `add-agent-docs`.

## Output

```
Done.
Updated: auth.md, data-model.md
Up to date (timestamp bumped): cron-jobs.md, conventions.md
Skipped: http-layer.md
Possibly undocumented (not created):
- stripe dep added → suggests billing.md
- features/reports/ created → suggests features/reports.md
Run add-agent-docs to create.
Flags: workflows.md changed — review AGENTS.md Quick Reference
```
