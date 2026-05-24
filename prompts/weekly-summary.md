---
description: Create a weekly summary from Obsidian daily notes (Monday–Sunday). Summarizes job hunt progress, learning, key wins, recurring mistakes, and improvement suggestions. Use when user asks for "weekly summary", "week summary", "ringkasan mingguan", or wants to reflect on the past week. Also triggers when user says "what did I do this week" or "summarize my week."
---

# Weekly Summary Generator

Creates a structured weekly summary from daily notes in this Obsidian vault. Pulls from daily notes (`calendar/daily-notes/`) and any notes created that week (`+/`, `atlas/`, `efforts/`).

## Workflow

### Step 1 — Determine Which Week to Summarize

The user's week runs **Monday to Sunday**.

Run this to get today's date and day of week:

```bash
date +"%Y-%m-%d %A %u"
```

Logic:

1. If today is **not Sunday** (day ≠ 7): the current week is unfinished.
   - Check if the **previous week** already has a summary at `calendar/weekly-summary/weekly-summary-{N}-{year}.md`.
   - If it does **not** exist, create the summary for the **previous week**.
   - If it already exists, respond: `Current week is not finished. Previous week already has a summary.`
2. If today **is Sunday** (day = 7): create the summary for the **current week**.

**Week number**: Week 1 is the Monday–Sunday span containing January 1. Count forward from there.

To compute the week number for a given Monday date:

```bash
python3 -c "
from datetime import date, timedelta
monday = date.fromisoformat('$MONDAY_DATE')
jan1 = date(monday.year, 1, 1)
week1_monday = jan1 - timedelta(days=jan1.weekday())
delta = monday - week1_monday
print(delta.days // 7 + 1)
"
```

### Step 2 — Gather Notes for the Week

Find all daily notes in the target week (Mon–Sun):

```bash
ls calendar/daily-notes/ | grep -E "^($MON|$TUE|$WED|$THU|$FRI|$SAT|$SUN)\.md$"
```

Read each daily note. Focus on these three sections:

| Section       | What's in it                                                                                                                                                                                                 |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `## 🦮 Todos` | Daily schedule with checkboxes: shadowing, grammar practice, episoden, SD2 (FE/BE interview prep), Leetcode, free learn, company applications. Also sub-items like `### Monday & Friday` for periodic tasks. |
| `## ☑️ Notes` | Freeform notes, reflections, deep dives, code reviews, system design breakdowns.                                                                                                                             |
| `## 💡 TIL`   | Quick bullet points — things learned that day.                                                                                                                                                               |

Also find notes created during the week:

```bash
find +/ atlas/ efforts/ -name "*.md" -newermt "$MONDAY" ! -newermt "$NEXT_MONDAY" 2>/dev/null
```

### Step 3 — Handle `###` Extraction from Notes and TIL

For each daily note, scan `## ☑️ Notes` and `## 💡 TIL` for `### ` headings (level-3). **Do not extract from `## 🦮 Todos`** — those sub-headings are part of the schedule template.

If you find an `### ` heading with substantial content (more than 2 lines):

1. Create a new note file in `+/` using the heading text as filename (e.g., `### Hardest problem distributed consensus and Raft` → `+/Hardest problem distributed consensus and Raft.md`).
2. Move the heading and its content into the new file. At the bottom, add `## Related\n- [[daily-note-filename]]` linking back to the source daily note.
3. In the daily note, replace the `### ` block with a wikilink: `- [[New Note Name]]` and a one-line summary.
4. Skip trivial `###` sections (2 lines or fewer).

**Before creating any extracted notes, tell the user which notes you plan to extract and ask for confirmation.**

### Step 4 — Generate the Weekly Summary

Create the file at `calendar/weekly-summary/weekly-summary-{N}-{year}.md`.

Use this exact structure:

```markdown
# Weekly Summary — Week {N}, {year}

**Period:** {Month Day}–{Day}, {year}

---

## 🗓️ Overview

Week period, main focus/theme, 1 or 2 additional overview points you think necessary.

## 💼 Job Hunt Progress

Summary of job-finding activities from the week. Pull from:

- `## 🦮 Todos` checkbox completion patterns — which blocks were consistently checked vs. skipped
- English practice: shadowing (speaking), grammar practice, episoden (conversation) — note completion rate
- DSA / System Design: Leetcode and SD2 blocks — completion and any content in Notes or TIL about these topics
- Company applications: "currates 3 companies & apply" block
- Any self-reflections in `## ☑️ Notes` about interview skills, speaking confidence, or technical prep
- Notable milestones or struggles in the job hunt

## 📚 Learning & Development

Things learned this week — from `## 💡 TIL`, `## ☑️ Notes`, and any notes created.

- Add comments or elaboration to TIL points — clarify if something is wrong, add context.
- Include notes created this week with wikilinks and short summaries.
- Group related learnings where it makes sense (e.g., all English grammar points together, all distributed systems concepts together).

## 🏆 Key Wins This Week

What actually clicked this week — the most meaningful takeaways. Pull from TIL, Notes, and completed Todos. Keep it short, highlight what mattered most, not a list of everything.

## ⚠️ Recurring Mistakes & Weak Spots

Be honest. Look across the week for patterns:

- Consistently unchecked Todos blocks — what's being avoided?
- If Notes or TIL show the same concept being revisited without progress, call it out
- Any self-critical reflections in Notes
- Misunderstandings visible in TIL entries (e.g., a concept described incorrectly)

Group by topic if there are multiple patterns. Don't sugarcoat — be specific.

## 💡 Honest Improvement Suggestions

Based on what was learned and the mistakes above, give direct, actionable suggestions:

- A different approach or method if the current one seems inefficient
- Specific topics or gaps to prioritize next week
- Habits or practice patterns worth adjusting
- Schedule changes if Todos patterns show systematic avoidance

Be direct. "You're avoiding X" is better than vague encouragement.

## 🔏 Notes Created

- List of notes created this week, with a brief summary for each.
- If you extracted any `###` sections into new notes in Step 3, list them here with wikilinks and descriptions.
```

### Content Guidelines

- **Conciseness**: Bullet points and short sentences. This is a reference, not a narrative.
- **Tone**: Professional but personal. Use "you" — "you wrote...", "you avoided...".
- **Honesty**: Don't sugarcoat. The user explicitly wants direct, useful feedback.
- **Emotions**: Note any emotional tone shifts in the writing — frustration, excitement, avoidance, confidence.
- **Links**: Use Obsidian wikilinks (`[[Note Name]]`) for referenced notes.
- **Empty sections**: If a section has no content for the week, write a single line noting that (e.g., "No notes created this week."). Don't remove the section.

### Step 5 — Stop

Once the summary file is created (and any `###` extractions are done), stop. Do not provide an inline summary in the terminal. Just confirm the file path.

## Vault Reference

- Daily notes: `calendar/daily-notes/YYYY-MM-DD.md`
- Weekly summaries: `calendar/weekly-summary/weekly-summary-{N}-{year}.md`
- Created notes: in existing folders.
- Daily note frontmatter is YAML between `---` markers — skip it when reading content
- The `## 🦮 Todos` schedule blocks are: shadowing, grammar practices, episoden, SD2 (FE+BE interview prep), Leetcode, free learn, company applications, Familya, Day Recap, and occasionally `### Monday & Friday` for Quran study
