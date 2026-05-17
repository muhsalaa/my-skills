---
name: update-docs
description: Refresh existing docs/handbook/ files against code changes since each doc's last_updated date. Two modes — auto (scan every doc, default when no args) and manual (user names specific docs). Uses git log committed-only diff from each doc's last_updated date, maps docs to code paths via doc-body grep + catalog signals + per-feature folder match, performs whole-doc auto-edit against current scan (except ADRs — body preserved, status flip only), bumps last_updated to today. Never touches docs/handbook/index.md, root README.md / CONTRIBUTING.md, .vitepress/config.ts, or docs/handbook/package.json. Use when user wants to refresh handbook, sync docs/handbook to recent code changes, says "update handbook", "refresh handbook", "/update-docs", or names docs to refresh ("update auth.md" when docs/handbook/auth.md exists).
---

# Update Docs

Refreshes `docs/handbook/*.md` against code changes since each file's `last_updated` frontmatter date. Whole-doc auto-edit for regular docs; ADRs are lifecycle-managed (frontmatter flip + scaffold new supersede). Committed-only diff.

## Self-contained

This skill is standalone. It does not depend on init-docs, add-docs, or agent-side siblings. Signal path-mapping reference is duplicated in [CATALOG.md](CATALOG.md); keep in sync with update-agent-docs/CATALOG.md when either is updated.

Handbook is its own source of truth after init. Never re-cross-check `docs/agent/` during update — that was a one-time seed at init time.

## Flow

1. **Check `docs/handbook/` exists.** If missing, stop. Tell user to run `init-docs` first.

2. **Pick the doc set.**
   - **No args** (default = auto): list every `.md` under `docs/handbook/` (excluding `README.md` hubs and `index.md`) with its `last_updated`. Then confirm:
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

7. **Per-doc handling.**

   | Doc kind | Behavior |
   |----------|----------|
   | `design-decisions/NNNN-*.md` | **Never edit body.** Scan the slug subject against current manifest/scan. If subject absent (e.g., `0003-use-zeptomail.md` but deps now show `sendgrid`) → flag as potentially superseded. Offer per-ADR: **(1)** flip frontmatter `status: accepted` → `superseded` (add `superseded_by: NNNN-<new-slug>` once written); **(2)** scaffold new `NNNN-<migrate-slug>.md` with `status: accepted`, `supersedes: <old>`, Context auto-filled from scan diff, Rationale `TBD — fill in when the chooser is available.` User confirms per-ADR. No auto-flip. |
   | `getting-started.md` | If root `README.md` has been modified since `last_updated` → auto re-summarize opening and install/run section from the current README. Otherwise standard whole-doc auto-edit based on mapped code diff. |
   | `contributing.md` | If root `CONTRIBUTING.md` (or `.github/CONTRIBUTING.md`, `docs/CONTRIBUTING.md`) has been modified since `last_updated` → auto re-summarize from the current file, preserve the link-back. Otherwise standard whole-doc auto-edit. |
   | `glossary.md` | Add new bounded-context names detected since `last_updated` as stub entries with `**TODO:**` definitions. Keep existing entries untouched. |
   | `for-llms.md` | If `docs/agent/` structure changed (new docs, removed docs, `AGENTS.md` added/removed) → refresh the pointer list. No voice changes. |
   | `features/<name>.md` | Whole-doc auto-edit against current `features/<name>/` (or equivalent) scan. Opening story paragraph preserved unless diff contradicts it. |
   | Any other regular doc | Whole-doc auto-edit against current scan. Rewrite factual content to match code. Short narrative opening preserved when it still holds. |
   | Doc with 0 relevant commits | Report `up to date`, bump `last_updated` to today, move on. Bumping shortens the next run's diff window. |

   **Confirm per doc in auto mode** — `update auth.md? (y/n/skip)`. Bulk-yes is the user's call.

   **After any edit (or on up-to-date): bump `last_updated` to today.**

8. **Detect undocumented areas.** Scan the full diff window (all commits since earliest `last_updated` in the set), not just paths mapped to existing docs. Flag signals that have **no matching handbook doc**:
   - New manifest deps with catalog-known purpose (e.g., `stripe`, `bullmq`, `sentry`) and no doc covers it (no `billing.md` / `background-jobs.md` / `telemetry.md`).
   - New top-level feature folders (`features/<new>/`, `modules/<new>/`, `apps/<new>/`, `packages/<new>/`) with no `features/<new>.md`.
   - New cross-cutting folders (`cron/`, `queue/`, `webhooks/`, `adapters/`, `migrations/`) with no matching doc.
   - New infra surface (first `Dockerfile`, first `.github/workflows/*`, first `terraform/`) with no `deployment.md` / `ci-cd.md`.

   If any found, print after per-doc work:
   ```
   Possibly undocumented (found in diff, no matching doc):
   - stripe dep added → suggests billing.md / payments.md
   - features/reports/ created → suggests features/reports.md
   - .github/workflows/ added → suggests ci-cd.md

   Create new docs for these? (y/n)
   ```
   On yes: tell user to run `add-docs` with the suggested slugs. **Do not create docs here** — `add-docs` owns additions. No auto-invoke.

9. **Flag handbook drift (never auto-fix).** Print warnings at end, do not edit:
   - `package.json` `name` differs from `.vitepress/config.ts` `title` (strip ` · Handbook` suffix) → `⚠ Handbook title drift: package.json says "<x>", config.ts says "<y>". Review manually.`
   - Detected project package manager differs from the pm referenced in `docs/handbook/README.md` run instructions → `⚠ Package manager drift: project uses <pm-a>, handbook README references <pm-b>. Review manually.`

10. **Never touch.**
    - `docs/handbook/index.md` (VitePress homepage — curated)
    - Root `README.md`, `CONTRIBUTING.md`, any file outside `docs/handbook/`
    - `docs/handbook/.vitepress/config.ts` (sidebar/config — user territory; add-docs owns additions, update-docs stays out)
    - `docs/handbook/package.json` (reader deps)
    - ADR bodies (`design-decisions/NNNN-*.md`) — only frontmatter `status` flip allowed, scaffolding new ADR is permitted

11. **Never re-cross-check `docs/agent/`.** Handbook is self-sufficient after init. Any `docs/agent/` staleness is update-agent-docs' job.

## Constraints

- Committed only. Dirty working tree ignored — tell user to commit first if they want those changes considered.
- Whole-doc auto-edit for regular docs. Narrative prose may be rewritten; user preserves via git if needed.
- ADR bodies are preserved. Lifecycle actions: status flip + new ADR scaffold. Never rewrite rationale / consequences / context of an existing ADR.
- Always bump `last_updated` after a doc is processed (whether changed or up-to-date) — shrinks future diff windows.
- Per-doc confirm in auto mode.
- Never delete docs. Removed code → leave the doc; user decides whether to delete.
- Never create new docs. Undocumented areas are surfaced as suggestions only — hand off to `add-docs`.
- `last_updated` uses today's date from system, not memory.

## Output

```
Done.
Updated: auth.md, data-model.md, workflows.md
Up to date (timestamp bumped): cron-jobs.md, conventions.md
Skipped: http-layer.md

ADR lifecycle:
- design-decisions/0003-use-zeptomail.md → flipped to superseded
- design-decisions/0008-use-sendgrid.md → scaffolded (Rationale: TBD)

Possibly undocumented (not created):
- stripe dep added → suggests billing.md
- features/reports/ created → suggests features/reports.md
Run add-docs to create.

Flags (not auto-fixed):
- ⚠ Handbook title drift: package.json says "acme-app", config.ts says "AcmeApp · Handbook"
- ⚠ Package manager drift: project uses pnpm, handbook README references npm

Next:
  cd docs/handbook && <detected-pm> run dev
```
