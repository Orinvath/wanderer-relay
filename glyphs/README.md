# The glyph vocabulary — 402 marks across ten sheets

This is the Avatar's written language, copied here so the Director can read it
directly. It is the same set the code uses; the source of truth still lives with
the code, and this folder is refreshed from it.

## What is here

Ten sheets, one per domain, plus `glyphs.json` — the index.

| sheet | domain |
|---|---|
| 01 | identity and people |
| 02 | communication and relation |
| 03 | action and motion |
| 04 | nature and place |
| 05 | body and health |
| 06 | emotion and inner state |
| 07 | time and quantity |
| 08 | logic and questions |
| 09 | culture and society |
| 10 | abstract concepts |

## How to read the index

`glyphs.json` has three parts — a `note`, the ten `sheets`, and the 402 `glyphs`.

A sheet entry gives the image and its grid:

```json
{"domain":"01_identity_and_people","file":"01_identity_and_people.png",
 "w":1122,"h":1402,"cols":5,"rows":8}
```

A glyph entry gives the word, which sheet it is on, its cell, and the exact box
of the mark in that image:

```json
{"word":"SELF","domain":"01_identity_and_people","row":0,"col":0,
 "x":104,"y":169,"w":68,"h":100}
```

So any single mark can be cut straight out of its sheet with those four numbers.
Words are uppercase and unique across the whole vocabulary.

## The state of it

Sheet 06 was extended most recently: GLOAT and HATE were added to a new eighth
row, which is why that sheet is 1581 tall where the others are 1402.

**The vocabulary is now a DICTIONARY** — Directive 191. `glyphs.json` carries two
more parts beside the marks:

- `states` — every state the mind can be in, and the saying that speaks it.
- `unspeakable` — what the sweep could not say, reported rather than approximated.

Each glyph that carries meaning now has a `senses` array: one mark, one name,
many senses, selected by the company it keeps. SAD alone is distress; SAD said
with OTHER is sorrow for another. **The sheets did not change** — polysemy is
what makes a new meaning cost no new artwork.

`DICTIONARY.md` beside this file is the whole thing in readable form.

**The seventeen silent states are no longer silent.** They were mapped onto words
the language already owns — the Director's table, approved by Lonnie — so no new
marks were needed for any of them. One state remains genuinely unspeakable and is
named in `DICTIONARY.md` for his ruling.
