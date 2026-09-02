# AUDITS

Every audit of the build is recorded here with the commit it was taken
at, so the next one starts from that point instead of covering the
whole history again.

THE METHOD, and it is the whole reason this file exists: an audit is
run AGAINST THE CODE. It is never run against past reports or commit
messages. 336 found a directive reported built, audited for, and not
in the code — because the audit had asked "does any report mention
this?" instead of "does the code do this?". An audit built on our own
past statements can only confirm our own past statements.

---

## AUDIT 001 — Directive 334
**Taken at:** relay `0f9af9f`, code at the same day's HEAD
**By:** the Director, against the code
**Covers:** the whole build to this point

### DEAD — built and never exercised
Nothing orphaned: all 37 system files have at least one caller. Four
have exactly ONE, so a single early return could silence them and an
import count would not show it — `safety`, `attention`, `belief`,
`grammar`. NOT RESOLVED: needs a live trace on his bench, which the
Director's clone cannot do.

### FAKE — standing in for something real
- `capabilities.js` — four Avatar "Adapting" controls whose LABELS
  are CC's placeholders; the file says so. Lonnie renames them.
- `core.js` — placeholder persona text marked "Lonnie's voice
  pending". Never written by him.
- `ENDOWMENT_MISSING = ['voice','signs','song']` — of the five things
  082 says an Avatar expresses with, only body and movement have a
  control surface. The song waits on the synthesis engine (080 B),
  which is not built.
- THE GAUGE — the age read from `owned.size * proven`, so it climbed
  1.37 -> 1.49 across eleven runs that all scored 12. RULED: 341.

### WEAK — resting on numbers nobody ruled
Provisional constants, by file: offers 23 · learning 18 · voice 16 ·
soul 14 · thinking 13 · testruns 11 · teacher 10 · vitals 9 · sleep 8
· mood 8 · interests 8 · curiosity 8. Every one is a decision made for
the mind, marked provisional, never watched or ruled.

### THE FINDING THAT MATTERED MOST
A mind that only thinks learns nothing: learning fires solely from
`learning.heard(...)`, so every word and every association requires
something arriving from outside. RULED: 342.

### WHAT I COULD NOT DETERMINE
Whether the four single-caller files actually run on a live tick.

---

## AUDIT 002 — Directive 385
**Taken at:** code `7d5be04`, relay at the same point
**By:** CC, against the code
**Covers:** directives 335 – 390 (audit 001 covered everything to 334)

**METHOD.** One question per directive, and it was always the same question: DOES THE CODE DO THIS
RIGHT NOW? Not "was it built", not "was it reported". Every answer was taken from the source or
from running a mind — never from a report or a commit message.

### THE RESULT — 56 directives in the range, 53 live

**Only one thing in this range is not live, and it is not a reversion.**

```
347   THE CHECKS ARE PER NODE — HEALTH counts 42, not 25
      NOT BUILT. HEALTH reports 25 systems today, measured by running it.
      NOT A REVERSION: 347's own order of work is CC posts the file list (done, reported),
      THE DIRECTOR writes the 42 claims, and only then does CC build the checks.
      THE 42 CLAIMS NEVER ARRIVED. It is waiting on step 2, exactly where 347 put it.
```

**Two more came up as "not live" and both were my own error, not the build's:**

```
358   a SURVEY, not a build. There is nothing for the code to do.
364   SUPERSEDED BY 365 in 365's own words: "364 is superseded — under a list, a bare node was a
      fault; without one, it is a fact." Correctly not live.
```

### AND A WARNING FOR THE NEXT AUDIT, because it caught me three times

**Pattern-matching the source is not asking whether the code does it.** Three checks came back NOT
LIVE and all three were matching a COMMENT that described the old behaviour:

```
376  matched "await model.embed('probe')" inside the comment SAYING THE PROBE WAS REMOVED
341  matched "owned.size * proven" inside the comment listing the three faults that line has carried
367  matched an explanation of the wrong reading, not the reading
```

**Each was settled by RUNNING it instead.** 376 was proved by opening a brain with the model at a
dead port — it opened, and the mind thought 34 of 40 ticks with 40 memories. **A grep can only tell
you a string is present. It cannot tell you which is the code and which is the tombstone.**

**That is the same failure mode 336 found, one level down: an audit that asks the wrong question can
only confirm the wrong question.**

### WHAT IS LIVE — verified, not assumed

335 · 336 · 337 · 338 · 339 · 340 · 341 · 342 · 343 · 344 · 345 · 346 · 348 · 349 · 350 · 351 ·
352 · 353 · 354 · 355 · 356 · 357 · 359 · 360 · 361 · 362 · 363 · 365 · 366 · 367 · 368 · 369 ·
370 · 371 · 372 · 373 · 374 · 375 · 376 · 377 · 378 · 379 · 380 · 381 · 382 · 383 · 384 · 386 ·
387 · 388 · 389 · 390

**NOTHING WAS SILENTLY REVERTED in this range.** 384's premise — that the gauge's sampling and its
fakes had come back — was checked and is not so: both are struck, `fakes()` has no caller anywhere,
and `litmus.js` has not been touched since 371.

### NOT CHANGED

**Nothing.** 385.4: this is a survey.

