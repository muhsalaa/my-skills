# My Skills

A collection of my agent skills for I used for managing projects.

## Skills

| Skill | What it does |
|-------|-------------|
| **init-docs** | Initialize human-facing onboarding docs (`docs/handbook/`) for a codebase. Scans the project, proposes a doc structure, interviews you for narrative and rationale, and optionally installs a local VitePress reader. |
| **add-docs** | Add a single new doc to an existing `docs/handbook/`. Runs a grill-me interview, generates the file with frontmatter, and wires it into the VitePress sidebar and hub READMEs. |
| **update-docs** | Refresh existing `docs/handbook/` files against code changes since each doc's `last_updated`. Committed-only diff, whole-doc auto-edit, and ADR lifecycle management. |
| **init-agent-docs** | Initialize agent-friendly documentation (`docs/agent/` + `AGENTS.md`) for a codebase. Signal-driven, tailored to your stack, with progressive disclosure so agents load only what they need. |
| **add-agent-docs** | Add a single new doc to an existing `docs/agent/`. Runs a grill-me interview, generates the file with frontmatter, and appends links to the hub README and `AGENTS.md`. |
| **update-agent-docs** | Refresh existing `docs/agent/` files against code changes since each doc's `last_updated`. Committed-only diff, surgical edits, and flags undocumented areas. |
| **validate-docs** | Fact-check documentation against the codebase. Extracts technical claims (paths, packages, commands, routes, env vars) and verifies them via filesystem, manifest lookup, or grep. Read-only — terminal report only. |
| **own-this** | Help a developer understand an unfamiliar codebase well enough to contribute fast. Topic-filtered mode supported. Use when saying "own-this", "help me understand this repo", "I inherited this code", or anything expressing unfamiliarity with a codebase. |
