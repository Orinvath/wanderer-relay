# THE BOARD — everything left undone

Organised as he asked: WHAT SYSTEM · WHAT NODE IT TOUCHES · WHEN IT
WAS PLANNED. Nothing here is scheduled; this is so he can decide where
his time goes.

Kept current from here on. TODO.md holds the reasoning behind each
item; this is the index.

---

## A · WAITING ON HIS WORD — nothing moves until he rules

| # | What | System | Node | Planned |
|---|---|---|---|---|
| A1 | The three meter numbers: how many wrong ticks is a halt (60 now), how much slower recovery is than the climb (3x now), where the bar changes colour | health / nervous | needs.js | 440, today |
| A2 | Does the meter go on the other 41 nodes, now that one is proven | health / nervous | all 42 | 439, today |
| A3 | Chronic wrongness narrowing what it thinks about. Director researched it and RECOMMENDED AGAINST — effect is dementia over years, not global cognition (OR 0.99), and our bugs would leave permanent marks on the being | nervous | curiosity.js | 436.B.4, today |
| A4 | Red means two things on the Meaning Map: a word not yet known, and failure (349) | display | Meaning Map | this week |
| A5 | Collapsible panels — planned, not built | display | bench page | this week |
| A6 | The app's suite (phase 3) stops the mind's suites. ~3 min and 71 model calls a run; four of five standing reds are its, and all four print PASS text beside the red mark | suite | none — app | 417, this week |
| A7 | A clause whose honest bound would make it circular. Needs a ruling, not a repair | suite | unnamed | this week |
| A8 | His bench on 8793 is running old code; a permission gate blocked the restart | bench | — | today |

## B · ORDERED AND NOT YET BUILT

| # | What | System | Node | Planned |
|---|---|---|---|---|
| B1 | The 41 remaining node claims, one check per file, each with its forced-fail | health | all 42 | 428, today |
| B2 | The `/code-review` Stop hook, plus hooks blocking writes to geometry.js and language.js, and blocking unnamed commits | process | — | 442, today |
| B3 | Prune CLAUDE.md where a hook now enforces a law | process | — | 442, today |
| B4 | Backfill MIND_DECISIONS — 115 commits landed with no row | governance | all | 430, today |

## C · DESIGNED, NOT SCHEDULED — his to start when he wants

| # | What | System | Node | Planned |
|---|---|---|---|---|
| C1 | ABLATION: every node clickable, switchable, tweakable, with the science's predicted deficit written before each switch is flipped | health / display | all 42 | this week |
| C2 | The mind learns grammar — from the model's corrections, and taught the parts of speech, with a pick-out-the-noun test running during school | learning | grammar, teacher | this week |
| C3 | Mimicry and pattern: it learns the teacher's patterns, not only words; WHY as a part of curiosity | learning | curiosity, learning | this week |
| C4 | Check whether ASK can fire at all. Not called a fault — a two-year-old mind may be too young, and the science agrees. But a mind that CANNOT ask is broken, and both look identical from outside | thought | curiosity, voice | this week |
| C5 | The mind finishes its own language: when it knows all 402, it makes marks for words that have none. Form is HIS | language | glyphs | this week |
| C6 | Reproduce human failures — overextension, the vocabulary spurt, primacy/recency, source-monitoring failure. Each has a published curve | validation | learning, memory | this week |
| C7 | Small-world structure in the language space — clustering, short paths, hubs that are its earliest and commonest words. Measurable against SWOW | validation | language, geometry | this week |
| C8 | Tests with a right answer: a thing hidden and found later, a sequence done in order, a promise kept across a sleep | validation | thinking, memory | this week |
| C9 | An emoji node for expression. WHAT IT IS FOR is his to say before it is designed | display / body | new | this week |
| C10 | Log accidental validation when it happens — the vegetative-state finding was one | validation | — | this week |

## D · KNOWN GAPS, NO PLAN YET

| # | What | System | Node | Noted |
|---|---|---|---|---|
| D1 | No planning and no problem solving. The field's own column, empty here. His answer: address it through the language and right-answer tests, not by importing a planner | thought | — | this week |
| D2 | Sixteen app files live in this repository. The mind imports none of them | architecture | — | 417 |
| D3 | One instance, one bench, no external test. Stands until it works | validation | — | this week |

---

# E · AUDIT OF THE WHOLE PROJECT — findings, oldest first

**Method: checked AGAINST THE CODE, not against reports.** That is
AUDITS.md's own rule and the thing CC's earlier audit failed to do.
Every directive number was first checked for a report; only four of
436 have none (190 was cancelled by him; 423, 424 and 430 were sent
today). **So there is no backlog of forgotten directives.** What
follows is where the CODE does not match what was ruled.

Anything needing his running bench is FLAGGED, not claimed.

## E1 · 236 — A REPLAY MAY NOT RUN WHOLE
**Ruled:** an interrupted story STARTS OVER from the beginning, whole,
always. The mind must hold the WHOLE story to understand it.
**In the code:** `stories.js` around line 92 — when a life holds more
touching episodes than `STORY.RECALL` allows, it takes THE MOST RECENT
RUN of them. CC wrote the tension down deliberately rather than hiding
it, and said the distinction is his to change.
**So a replay is whole only while the episodes fit the capacity.**
Unruled.

## E2 · 319 / 339 — THE MOUTH IS STILL CAPPED
**Ruled (339):** NO CAPACITY MAY CAP WHAT THE MIND CHOOSES TO DO. The
line is as long as the mind can honestly build from what it has heard,
and stops when IT stops. Age staging was struck at 319.
**In the code:** `growth.js:90` — `runWords: born 9, adult 9` and
`gramDepth: born 4, adult 4`. Born equals adult, so AGE no longer
lifts them — but NINE IS STILL A CEILING, and `voice.js:610` and
`:612` pass both into every line the mind builds.
**A mind that has heard fourteen-word sentences may still say nine.**
The file's own comment says "the ceiling is one line away in this
table."
**This is the fault he has been describing for days in another form.**

## E3 · 372 — ONE GLYPH FALLBACK SURVIVES
**Ruled:** thinking draws from what the mind KNOWS, not from
`glyphs.js`. The filter came out of five places.
**In the code:** `thinking.js:701` — `known.length ? known :
WORDS.filter(...)`. When the mind knows nothing yet, its comparison
words STILL COME FROM THE 402. One line, and it only bites on a young
mind — which is every mind he has tested.
(`thinking.js:233`, the domain draw, is the one 372.5 expressly
allowed and is NOT a fault.)

## E4 · 426 — THE FELT TRAIL WAS NEVER DIAGNOSED
**Ordered:** find why every story beat reads `[0,0,0]`, and say
whether the first story gate has been refusing every self-told story
or passing them all.
**Status: no report.** The machinery is present (`storygates.js:37`
`wasFelt`, called by the gate at :103), so this needs his bench rather
than the code. Still open.

## E5 · 429 — THE TEACHER'S LINE KIND IS NOT MARKED
**Ruled:** every teacher line in the chat log is marked NAMING or
STORY. CC built it, then REVERTED it because it had built on a
question rather than an instruction, and reported the gap instead.
Correct behaviour; the work is not done.

## WHAT THE AUDIT CONFIRMED AS HONOURED IN THE CODE
013 (the two deletes are a host's revocation and a checkpoint restore,
both lawful) · 051 · 062 · 236's own check · 305 · 321 (a load resumes
`livedHours`) · 354 (no sample) · 360 (no fakes) · 365 (no edge list) ·
373 · 376 (the probe is gone; both mentions are comments recording it)
· 381 (mismatch and pertinence are in `attention.js`, under different
words than the directive used) · 397 · 406 · 409 · 415 (reverted
clean) · 419 · 436 · 440.

## COVERAGE, HONESTLY
This pass checked every directive for a report, and then checked the
BEHAVIOURAL directives against the code by hand. Process directives —
how we work, how reports are written — have no code to check and were
not audited. A second pass over the older behavioural directives
(001-200) is worth doing and has not been done.
