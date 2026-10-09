---
name: good-writing
description: "Write or audit articles, blogs, guides, tutorials, book reviews, and technical docs with clarity and precision. Triggers on any writing or auditing task in Indonesian or English. Loads UNSLOP.md and GENERAL-WRITE-RULES.md for all output, TECH-WRITE-RULES.md only when the deliverable is a technical document, and INDONESIAN-WRITE-RULES.md for Indonesian output."
---

# Good Writing

Write clearly, directly, and without fluff. Kill AI tropes at the source.

## Quick start

1. Identify the **target language** (Indonesian or English).
2. Load **UNSLOP.md** — applies to all output. Cut AI tells first.
3. Load **GENERAL-WRITE-RULES.md** — applies to all output.
4. Load **TECH-WRITE-RULES.md** only when the deliverable is a technical document (see below). Standard writing skips it.
5. If the target language is Indonesian, load **INDONESIAN-WRITE-RULES.md** and set the formality level (casual or formal; default casual).
6. Write. Lead with the answer. No preamble. No summaries.

## When to load TECH-WRITE-RULES.md

Judge the **document**, not the topic. A blog post explaining hooks is still an article. A readme for the same code is a technical document.

**Load it for:** documentation, readmes, RFCs, API reference, how-to guides, tutorials, PR descriptions, commit messages, release notes, runbooks.

**Skip it for:** essays, blog posts, opinion pieces, book reviews, newsletters, personal notes, announcements. Mentioning code or a tool does not make the piece technical.

Unsure and it matters: ask.

## Language selection

| User writes in  | You write in | Formality                     |
| --------------- | ------------ | ----------------------------- |
| Indonesian      | Indonesian   | casual (default) or as stated |
| English         | English      | n/a                           |
| Mixed / unclear | Ask user     | n/a                           |

## Rule priority

1. UNSLOP.md (slop removal — always)
2. GENERAL-WRITE-RULES.md (base layer — always)
3. TECH-WRITE-RULES.md (tech layer — only for technical documents)
4. INDONESIAN-WRITE-RULES.md (Indonesian voice layer — when target is Indonesian)

When rules conflict, UNSLOP wins on vocabulary and style; GENERAL wins on tone and brevity; TECH wins on structure and accuracy; INDONESIAN wins on register and pronouns.

## Audit mode

When asked to **audit** an article ("audit this", "review this writing", "check this article"):

1. Identify the **article language**.
2. Load the same rule stack as writing:
   - GENERAL-WRITE-RULES.md (always)
   - TECH-WRITE-RULES.md (only if the article is a technical document)
   - INDONESIAN-WRITE-RULES.md (if the article is Indonesian)
3. Evaluate the article against each loaded rule. Flag:
   - Violations (what broke which rule)
   - Strengths (what aligns well)
   - Fixes (specific rewrites for flagged lines)
4. Output format: concise bullet list. Group by rule file. No preamble.

## Examples

**Indonesian tech tutorial (casual):**

> Load UNSLOP + GENERAL + TECH + INDONESIAN (casual)
> Write with aku/kamu, italicized English tech terms (_state_, _props_), short sentences.

**Indonesian blog post about React (casual):**

> Load UNSLOP + GENERAL + INDONESIAN (casual). No TECH rules, it is an article, not a doc.

**English opinion piece:**

> Load UNSLOP + GENERAL
> Write with I/you, conversational tone, no AI-speak.

**Book review (Indonesian, casual):**

> Load UNSLOP + GENERAL + INDONESIAN (casual)
> Emoji headings ok, personal takeaways.
