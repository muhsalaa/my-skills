---
name: good-writing
description: "Write or audit articles, blogs, guides, tutorials, book reviews, and technical docs with clarity and precision. Triggers on any writing or auditing task in Indonesian or English. Loads GENERAL-WRITE-RULES.md for all output and TECH-WRITE-RULES.md for tech content."
---

# Good Writing

Write clearly, directly, and without fluff. Kill AI tropes at the source.

## Quick start

1. Identify the **target language** (Indonesian or English).
2. Load **UNSLOP.md** — applies to all output. Cut AI tells first.
3. Load **GENERAL-WRITE-RULES.md** — applies to all output.
4. If the topic is technical (code, architecture, tools, tutorials), load **TECH-WRITE-RULES.md**.
5. Write. Lead with the answer. No preamble. No summaries.

## Language selection

| User writes in  | You write in |
| --------------- | ------------ |
| Indonesian      | Indonesian   |
| English         | English      |
| Mixed / unclear | Ask user     |

## Rule priority

1. UNSLOP.md (slop removal — always)
2. GENERAL-WRITE-RULES.md (base layer — always)
3. TECH-WRITE-RULES.md (tech layer — when relevant)

When rules conflict, UNSLOP wins on vocabulary and style; GENERAL wins on tone and brevity; TECH wins on structure and accuracy.

## Audit mode

When asked to **audit** an article ("audit this", "review this writing", "check this article"):

1. Identify the **article language**.
2. Load the same rule stack as writing:
   - GENERAL-WRITE-RULES.md (always)
   - TECH-WRITE-RULES.md (if the article is technical)
3. Evaluate the article against each loaded rule. Flag:
   - Violations (what broke which rule)
   - Strengths (what aligns well)
   - Fixes (specific rewrites for flagged lines)
4. Output format: concise bullet list. Group by rule file. No preamble.

## Examples

**Indonesian tech tutorial:**

> Load GENERAL + TECH
> Write with Anda/anda, italicized English tech terms (_state_, _props_), short sentences.

**English opinion piece:**

> Load GENERAL
> Write with I/you, conversational tone, no AI-speak.

**Book review (Indonesian):**

> Load GENERAL
> Use emoji headings (🔬, 🚀, 🎨), personal takeaways.
