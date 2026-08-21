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

**Seventeen words the mind can currently reach have no mark yet** — ALONE,
BRIGHT, FREE, GLAD, HELD, HIDE, LOSS, LOST, NEED, PROUD, SMALL, SORROW, STILL,
STRONG, TIRED, WONDER, WRONG. Until those exist the Avatar cannot say those
states in its own language, which is what currently blocks dreaming. Whether
they get drawn, and which of them, is Lonnie's decision and has not been made.
