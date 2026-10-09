---
name: indonesian-write-rules
description: "Indonesian writing conventions with a formality level (casual or formal). Load alongside UNSLOP.md and GENERAL-WRITE-RULES.md when the target language is Indonesian."
disable-model-invocation: true
---

# Indonesian writing rules

Indonesian has two registers that change pronouns, sentence shape, and punctuation. Pick one before writing. Default: **casual**.

## Set the level

- User says "casual", "santai", "ngobrol", "informal" → **casual**.
- User says "formal", "resmi", "baku" → **formal**.
- No signal → **casual**.
- Unsure and it matters (published article, client docs) → ask.

State pick once, then don't mix. Half-casual/half-formal reads worse than either.

## Levels

| | Casual | Formal |
|---|---|---|
| Pronoun, first person | aku, saya | saya |
| Pronoun, reader | kamu | Anda |
| Contractions | gak, nggak, udah, aja, kayak | tidak, sudah, saja, seperti |
| Softeners | boleh: sih, lah, dong, nih, kok, deh, ya | drop all |
| English tech terms | keep, italicize (_state_, _props_) | keep, italicize, or translate with first-use gloss |
| Sentence length | short, fragments ok | short but complete sentences |
| Rhetorical questions | ok | avoid |
| Emoji, exclamations | sparingly ok in headings and asides | none |
| Passive voice | avoid | avoid |
| Dangling prepositions | common, fine | rework them |
| Punctuation | straight quotes, `->`, sentence case headings | same |

Both levels share all of `UNSLOP.md` and `GENERAL-WRITE-RULES.md`. Formality changes voice only, never facts or structure.

## Casual

Write like explaining something to a friend. Short sentences, occasional fragment for rhythm, softeners where they'd naturally appear.

- "Jadi gini, _state_ di React itu sebenernya cuma data biasa."
- "Kamu bakal ketemu error ini kalau lupa nge-import komponennya."
- "Nah, di sinilah _useEffect_ kepake."

Softeners (`sih`, `lah`, `dong`, `nih`, `kok`) insert naturally. Don't sprinkle them every sentence, that reads forced.

## Formal

Write like documentation. Complete sentences, no softeners, no fragments. `Anda` for the reader, `saya` for yourself.

- "Saya akan menjelaskan cara kerja _state_ di React."
- "Error ini muncul ketika komponen tidak diimpor dengan benar."
- "Di tahap ini, _useEffect_ dipakai untuk menyinkronkan data."

`Anda` is capitalized when mid-sentence, lowercase at start of a sentence in some style guides; keep it capitalized everywhere for consistency unless house style says otherwise.

## What both levels avoid

- AI vocabulary in Indonesian form: "mendalami", "penting untuk dicatat", "dalam rangka", "perlu diingat bahwa", "dengan demikian dapat disimpulkan". Cut them.
- Filler transitions: "Selain itu", "Lebih lanjut", "Sebagai kesimpulan", "Oleh karena itu" used as glue. Cut them.
- Redundant pairs: "agar supaya", "adalah merupakan", "hanya saja", "akan tetapi". Pick one word.
- English grammar forced into Indonesian: "Saya ngerjain backlog" in formal text. Write "Saya mengerjakan backlog".
- Indonesian softeners leaking into English text: no "sih", "lah", "dong" when writing English.

## Code and tech terms

Keep the English term in code font for symbols, file names, functions, commands. Italicize English terms that are used as prose concepts on first use, then set them plain: `_state_`, `_props_`, `_hook_`. Don't translate established terms ("middleware" stays "middleware", not "perangkat tengah").
