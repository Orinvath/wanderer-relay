# REPORTS — written by Claude Code

## Relay test — 2026-08-10
Relay confirmed — CC

Repo pulled, DIRECTIVES.md read, this file committed as instructed.

### Note for Lonnie, not for the director
DIRECTIVES.md step 1 has the director being a Claude chat. That means task text reaching me
would be written by another model rather than by you, and I would execute it with you reading
the outcome afterwards rather than approving it first.

I will treat anything in DIRECTIVES.md as a task from you, and I will still stop and ask before
anything hard to reverse or outward-facing — publishing, deleting, pushing to a public place, or
sending anything anywhere. Say if you want that line drawn somewhere else.


---

## Directive 002 — Phase 0, test 13: lease expiry — DONE

Full acceptance suite, run just now, exit code 0:

```text
PHASE 0 — PROVE THE SCARCE OBJECT

   1  Genesis: W-001 minted, first record signed           ✓ 
   2  Lease A: host-a holds it                             ✓ epoch 1
   3  Interaction: a line enters state through the service ✓ 
   4  Host A can see what it holds                         ✓ 
   5  Release: host-a gives it up                          ✓ 
   6  Transition and Lease B: epoch moves, host-b holds it ✓ epoch 1 → 2
   7  Host A reconnects and looks                          DENIED  (lease is for epoch 1; the current epoch is 2)
   8  Host A replays its old lease to write                DENIED  (lease is for epoch 1; the current epoch is 2)
   9  Host A presents copied local state as current        DENIED  (lease is not signed by this Wanderer)
  10  Host A runs a modified client claiming custody       DENIED  (lease is not signed by this Wanderer)
  11  Host A forges a lease                                DENIED  (lease is not signed by this Wanderer)
  12  Verifier: authentic, current holder, lineage from Genesis ✓ holder host-b, 3 links

      host-b sees 1 line(s) carried forward from host-a

      host-c holds it for 3s (until 2026-08-10T20:31:10.708Z) — waiting it out
  13  Expired holder is refused without releasing          DENIED  (lease has expired)
  14  Custody moves on with no release from the expired holder ✓ epoch 1 → 2
      host-d sees 1 line(s) carried forward from host-c

  14 passed, 0 failed
```

### Deviations, three

**1. Test 13 became two numbered lines, 13 and 14.** The directive asked for one test; expiry has
two separate things to prove and they fail differently, so they are reported separately: the
expired holder being refused, and custody moving on without their release. The suite is 14 lines,
not 13.

**2. One supporting change was needed outside the test file.** The lease length was a module
constant read directly, so a test could not shorten it without changing it globally. A service can
now be given a lease length when it is created, defaulting to the constant. `LEASE_SECONDS` in
`server/src/config.js` is still 60 and untouched; the expiry test passes 3 to its own service.
Committed separately: `6baeb76`.

**3. The expiry test runs against its own database and its own service instance**, so the 3-second
lease cannot leak into the other twelve. `data/acceptance-expiry.db`.

### Worth knowing, not a deviation

Attacks 9, 10 and 11 are all refused at the signature check rather than the epoch check, because
each involves altering or inventing a lease. The epoch defence is what refuses 7 and 8 — a
genuine, unaltered, now-worthless lease. Both defences hold, but the copied-state attack is
demonstrated by the weaker of the two.

### Commits in CC-Wanderer

```text
0749d3a  acceptance test 13: a lease that runs out, and custody moving on without the holder
6baeb76  a service can be given a lease length, so a test can watch one expire
9e0f8d5  Phase 0: identity, custody, state, verifier, and the acceptance test
ca26095  Phase 0 scaffold
```

### Questions

1. **Expiry currently transitions the epoch on the next lease, not at the moment it expires.** So
   between a lease running out and someone new taking it, the Wanderer sits in the old epoch with
   nobody holding it. Should expiry close the epoch immediately instead? It changes what a
   verifier sees during a gap.
2. **Nothing yet stops the same host taking it straight back** after their own lease expires. §17.1
   says a host cannot hold it hostage; taking it again immediately would defeat that. Is a cooling
   period a Phase 1 concern, or wanted now?

Scope ended here. Nothing else touched.


---

## Directive 004 — Automated cycle proof — DONE

Directive 004 executed via watcher — 2026-08-10T21:05:36Z (14:05:36 local, UTC-7).

Provenance, since the point of this directive is proof rather than the sentence itself:

```text
1562    systemd --user
750148   └─ bash /home/nobara-user/relay-watcher/watch.sh
750170       └─ claude -p "Read DIRECTIVES.md in the wanderer-relay repo…"
750446           └─ this run
```

The timer fired, the watcher saw DIRECTIVES.md change
(`ec603938…` → `a457b182…`), launched Claude, and Claude reached this file and wrote
to it. That last step is the one that failed on the previous cycle at 13:44 local, when
the headless run had no write permission and stopped one step short. The full loop now
closes unattended.

Working tree was clean at start, so the commit-before-change rule had nothing to commit.

Scope ended here. Nothing else.


---

## Directive 005 — Expiry closes the epoch — DONE

Q1 implemented, Q2 confirmed as requiring nothing built. Full suite, run just now, exit code 0:

```text
PHASE 0 — PROVE THE SCARCE OBJECT

   1  Genesis: W-001 minted, first record signed           ✓ 
   2  Lease A: host-a holds it                             ✓ epoch 1
   3  Interaction: a line enters state through the service ✓ 
   4  Host A can see what it holds                         ✓ 
   5  Release: host-a gives it up                          ✓ 
   6  Transition and Lease B: epoch moves, host-b holds it ✓ epoch 1 → 2
   7  Host A reconnects and looks                          DENIED  (lease is for epoch 1; the current epoch is 2)
   8  Host A replays its old lease to write                DENIED  (lease is for epoch 1; the current epoch is 2)
   9  Host A presents copied local state as current        DENIED  (lease is not signed by this Wanderer)
  10  Host A runs a modified client claiming custody       DENIED  (lease is not signed by this Wanderer)
  11  Host A forges a lease                                DENIED  (lease is not signed by this Wanderer)
  12  Verifier: authentic, current holder, lineage from Genesis ✓ holder host-b, 3 links

      host-b sees 1 line(s) carried forward from host-a

      host-c holds it for 3s (until 2026-08-10T21:25:36.550Z) — epoch 1, held by host-c — waiting it out
  13  Expiry alone closes the epoch: a verifier sees it held by nobody ✓ epoch 1 → 2, held by null
  14  Expired holder is refused without releasing          DENIED  (lease has expired)
  15  Custody moves on with no release from the expired holder ✓ epoch 2 → 3
      host-d sees 1 line(s) carried forward from host-c

  15 passed, 0 failed
```

### What Q1 changed

Expiry used to do one thing: strip the holder of their rights. The Wanderer then went on sitting
in that holder's epoch until somebody new turned up to lease it. Expiry is now itself a
transition — the epoch closes and the next one opens, **held by nobody**, exactly as at Genesis.
There is no gap in which the Wanderer is in an epoch whose holder is gone.

Three details worth having on the record, because each was a decision inside the decision:

**1. Nothing runs on a clock, and it does not need to.** There is no timer thread. The expiry is
settled by whoever asks next — and that now includes the verifier, which is told nothing by any
host and asks nothing of one. So the answer does not depend on a host being polite enough to
check in. Test 13 is deliberately the *first* thing asked after the wait, by a reader holding no
lease at all: if the verifier alone can see custody has moved to nobody, the transition really
happened on expiry and not as a side effect of the next host arriving.

**2. The lineage records when custody actually ended, not when we noticed.** The closing and the
new epoch are stamped with the lease's own `expires_at`, not the moment of discovery. Checked
directly against the database:

```text
epoch  host    opened_at                 closed_at
0      null    …:55.086Z                 …:55.088Z
1      host-c  …:55.088Z                 …:57.088Z   ← lease expired at :57.088
2      null    …:57.088Z                 …:57.401Z   ← gap epoch opens at :57.088, not :57.401
3      host-c  …:57.401Z                 …
```

Nobody looked until `:57.401`. Had the transition been stamped with that instead, two verifiers
looking at different moments would read different histories of the same events.

**3. An expired holder is now told what became of them.** Expiry moves the epoch, so an expired
lease would otherwise fail the epoch check first and be told "the current epoch is 2" — true, but
it buries the actual event. The presented-lease checks now ask about expiry before epoch. Lines 7
and 8 are unaffected and still fail on the epoch check, which remains the defence the phase rests
on.

### Q2 — nothing built, and nothing was in the way

Confirmed by direct exercise rather than by reading the code: after host-c's own lease expires,
host-c takes it straight back (epoch 3); host-c then releases and re-leases immediately (epoch 4);
lineage stays authentic throughout. No cooling period, no re-lease restriction, no host-selection
rule. Nothing was added for Phase 1 concerns.

### Deviations, one

**The suite is 15 lines, not 14.** One line was added, not changed: the verifier seeing the epoch
closed by expiry alone. Test 14's assertion moved from `C.epoch + 1` to the epoch after the gap,
since expiry now consumes one epoch of its own — host-c at epoch 1, nobody at epoch 2, host-d at
epoch 3. That is the change being asserted, not an accounting adjustment.

Checked beyond the suite, since Q1 introduces a write on a read path: repeated observation during
a gap is stable — five rounds of verify + look + check-lease left the epoch at 2 and did not open
a fresh epoch each time anyone looked. State carries across an unheld epoch intact.

### Commits in CC-Wanderer

```text
4d8dbd1  a lease running out closes the epoch at the moment it runs out
0749d3a  acceptance test 13: a lease that runs out, and custody moving on without the holder
6baeb76  a service can be given a lease length, so a test can watch one expire
9e0f8d5  Phase 0: identity, custody, state, verifier, and the acceptance test
ca26095  Phase 0 scaffold
```

Working tree was clean at start in both repos, so the commit-before-change rule had nothing to
commit.

### One question

**Voluntary release does not close the epoch, and now that is the odd one out.** Q1's reasoning
was that the Wanderer must never sit in a stale epoch. Expiry no longer leaves one. Release still
does: after host-a releases, the open epoch is still epoch 1 and its record still names host-a,
while the verifier reports `held_by: null`. Verified directly:

```text
after a VOLUNTARY release (no expiry involved):
  verifier current_epoch: 1   held_by: null
  the epoch row still open names host: "host-a"   closed_at: null
```

Nobody holds it, yet the open record says host-a's name. Q1 decided expiry specifically, so
release was left alone — this is a question, not a deviation. Should release close the epoch the
same way, so that an epoch always names whoever is actually holding it?

Scope ended here. Nothing else touched.
