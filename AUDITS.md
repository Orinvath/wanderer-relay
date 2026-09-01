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
