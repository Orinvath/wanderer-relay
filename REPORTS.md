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


---

## Directive 006 — Release closes the epoch too — DONE

Implemented. Full suite, run just now, exit code 0:

```text
PHASE 0 — PROVE THE SCARCE OBJECT

   1  Genesis: W-001 minted, first record signed           ✓ 
   2  Lease A: host-a holds it                             ✓ epoch 1
   3  Interaction: a line enters state through the service ✓ 
   4  Host A can see what it holds                         ✓ 
   5  Release: host-a gives it up                          ✓ 
   6  Release alone closes the epoch: a verifier sees it held by nobody ✓ epoch 1 → 2, held by null
   7  Transition and Lease B: epoch moves, host-b holds it ✓ epoch 2 → 3
   8  Host A reconnects and looks                          DENIED  (lease is for epoch 1; the current epoch is 3)
   9  Host A replays its old lease to write                DENIED  (lease is for epoch 1; the current epoch is 3)
  10  Host A presents copied local state as current        DENIED  (lease is not signed by this Wanderer)
  11  Host A runs a modified client claiming custody       DENIED  (lease is not signed by this Wanderer)
  12  Host A forges a lease                                DENIED  (lease is not signed by this Wanderer)
  13  Verifier: authentic, current holder, lineage from Genesis ✓ holder host-b, 4 links

      host-b sees 1 line(s) carried forward from host-a

      host-c holds it for 3s (until 2026-08-10T21:38:39.049Z) — epoch 1, held by host-c — waiting it out
  14  Expiry alone closes the epoch: a verifier sees it held by nobody ✓ epoch 1 → 2, held by null
  15  Expired holder is refused without releasing          DENIED  (lease has expired)
  16  Custody moves on with no release from the expired holder ✓ epoch 2 → 3
      host-d sees 1 line(s) carried forward from host-c

  16 passed, 0 failed
```

### The question from Directive 005, answered directly

That report ended by showing the odd case. Same check, re-run against the code as it now stands:

```text
after a VOLUNTARY release (no expiry involved):
  verifier current_epoch: 2   held_by: null
  the epoch row still open names host: null   closed_at: null
  release said: epoch 1 given up, epoch 2 now open
```

Before, the open row named `"host-a"` while the verifier reported `held_by: null`. Those two now
say the same thing, which was the whole of the decision.

### What changed

Release did one thing: mark the lease given up. The epoch went on standing open in the departed
host's name until somebody new arrived. Release is now a transition, exactly as expiry became one
in 005 — the epoch closes as the host lets go, and the next opens **held by nobody**.

The two paths are now the same act with different triggers, and the code says so: both close the
open epoch and open an unheld one through the single `openEpoch` that has always been the only
place custody changes hands. One difference of substance remains, and it is real rather than an
oversight: **an expiry is stamped with the lease's own `expires_at`, because custody ended before
anyone noticed; a release is stamped with the moment it arrives, because for a release those are
the same moment.** Both record when custody actually ended.

Checked against the database, a lineage containing both kinds of letting go:

```text
epoch  host      opened_at                 closed_at
0      null      …:05.052Z                 …:05.068Z
1      host-a    …:05.068Z                 …:05.075Z   ← released
2      null      …:05.075Z                 …:05.085Z
3      host-a    …:05.085Z                 …:05.088Z   ← same host again, released again
4      null      …:05.088Z                 …:05.092Z
5      host-a    …:05.092Z                 …:05.095Z
6      null      …:05.095Z                 …:05.097Z
7      host-e    …:05.097Z                 …:08.097Z   ← EXPIRED at :08.097
8      null      …:08.097Z                 …:08.708Z   ← opens at :08.097, not at :08.708 when noticed
9      host-f    …:08.708Z                 …:08.712Z
10     null      …:08.712Z
```

Every epoch names its actual holder, or nobody. `authentic true, problems 0` throughout.

### Two consequences worth having on the record

**1. A released host is now refused by the epoch check rather than the released check.** Release
moves the epoch, so a returning host is told `lease is for epoch 1; the current epoch is 3` before
anything looks at whether they handed it back. That is the same defence lines 8 and 9 have always
rested on, and it is the stronger of the two — but it does mean `checkLease`'s fifth test, "has it
been given up?", is now unreachable: a lease's epoch can only be the open one while that lease is
live, and both ways of ceasing to be live now move the epoch. **I left the line in place rather
than deleting it.** It costs nothing, and removing a defence because the ordering currently makes
it redundant is how orderings come to be depended on silently. Say if you would rather it go.

**2. Every handover now costs two epochs, not one.** A→B used to be epoch 1→2; it is now 1→2→3,
with 2 held by nobody for however long the gap lasts. That is the decision working as intended —
the unheld interval is a real thing that now has a real record — but epoch numbers are no longer a
count of visits, and anything later that reads them as one will be wrong.

### Deviations, one

**The suite is 16 lines, not 15.** One line added, line 6: a verifier, holding no lease and told
nothing by anybody, seeing the epoch closed by the release alone — asked before host-b arrives, so
what it sees cannot be the next host's doing. It is deliberately the same proof in the same words
as line 14 does for expiry, since the two behaviours are now the same behaviour. Line 7's
assertion moved from `A.epoch + 1` to the epoch after the gap, for the same reason line 15's did
in 005: the transition being asserted now consumes an epoch of its own.

### Checked beyond the suite

- **Re-releasing an already-released lease** is refused, as is looking with it.
- **Q2 from Directive 005 still holds** — no restriction was introduced by accident. host-a
  releases and takes it straight back, twice in a row (epochs 3 and 5 above), unimpeded.
- **State survives every unheld epoch.** host-f, ten epochs and two kinds of letting go after
  host-a wrote the line, still sees it.
- **Repeated observation during a gap is stable.** Five rounds of verify + look left the epoch
  where it was; looking does not open epochs, which matters because both closing paths write on
  what is otherwise a read.

### Commits in CC-Wanderer

```text
b2f3387  letting go closes the epoch, however a host lets go
4d8dbd1  a lease running out closes the epoch at the moment it runs out
0749d3a  acceptance test 13: a lease that runs out, and custody moving on without the holder
6baeb76  a service can be given a lease length, so a test can watch one expire
9e0f8d5  Phase 0: identity, custody, state, verifier, and the acceptance test
ca26095  Phase 0 scaffold
```

Working tree was clean at start in both repos, so the commit-before-change rule had nothing to
commit.

Scope ended here. Nothing else touched.

---

## Directive 007 — Phase 1 begins: Custody — PLAN ONLY, STOPPED FOR APPROVAL

**Executed:** 2026-08-10, manual invocation.

Directive 007 says: read the spec's Phase 1 requirements, write PHASE1_PLAN.md, commit it, copy
its full text here, and stop. No implementation until the plan is approved. That is what happened.
Nothing in CC-Wanderer changed except the addition of the plan file.

### Before anything

Both working trees clean at start, so the commit-before-change rule had nothing to commit — the
same as the last two directives.

### Baseline re-checked rather than assumed

The plan says Phase 0 is green at 16 lines, so I ran the suite instead of quoting the Directive 006
report for it. `node server/src/acceptance.js` — **16 passed, 0 failed**, unchanged.

### What was read

§67 Phase 0 and Phase 1; §14 custody; §15 custody credential; §16 host authentication; §17.1
custody expiration; §18 release; §19 and §19.1 forced departure and visit duration; §20 arrival;
§12.1 canonical state; §34 host number; §35 host certificate; §36 verifiable credentials; §39
server security; §40 failure and recovery; §41 state hashing; §48 host selection; §59 privacy
rights; §61 authority model; §62 trust model; §65-B and §65-C open decisions; §66.8 through
§66.19, in particular §66.9 (the Phase 0 attack list), §66.10 and §66.11 (canonical state and
custody acceptance criteria), §66.15 through §66.18. Also all 682 lines of the Phase 0
implementation, since the plan has to attach to what is actually there rather than to what the
spec imagines.

### The two findings the plan is built on

Worth stating separately from the plan text, because they are the reason it is shaped as it is.

**1. A host is a string.** `POST /lease {wanderer, host}` takes `host` as free text, believes it,
and signs it into the lineage. There is no account, so §15's first line — `Host authentication →
Custody granted` — has no first line. Expected: accounts are exactly what Phase 1 is for.

**2. A live lease is a bearer token.** This one I did not expect, and it is not visible in the
Phase 0 suite, because every attack there is run by host-a *after* host-a's lease has died.
`checkLease` asks whether the lease is genuine, issued, live and current. It never asks who is
presenting it. So a live lease, if it leaks, **is** custody — whoever holds the bytes holds the
Wanderer, and the service cannot tell them from the host it was issued to.

Phase 0 was right to ship both. Its success condition was that a copy cannot become the real one,
and both hold up completely against that. Neither holds up against Phase 1's success condition,
which names hostile hosts. So the plan's central change is that a lease must be presented *by the
account it names*, proving a live session in the same request: two independent things to steal
instead of one, and the second cannot be copied, because it rests on a passkey that does not leave
the authenticator.

### Deviations

None. The directive's four steps were followed exactly and stop at the plan.

One judgement call worth flagging: the plan proposes a new dependency (`@simplewebauthn/server`),
which Phase 0 deliberately did without — it was built on express, better-sqlite3 and Node's own
crypto and nothing else. Hand-rolling WebAuthn attestation parsing is exactly what §66.15 warns
about, so I have proposed the dependency rather than quietly taking it, and it is open question 3.

### Commits

```text
16c6c07  Phase 1 proposed: custody belongs to a person, and a lease stops being a bearer token
b2f3387  letting go closes the epoch, however a host lets go
```

### Stopped here

No implementation. Eight open questions below, account recovery first — it is the one that decides
how much of the rest can be built, because every way of giving an account back to someone is also
a way of taking someone else's.

---

## PHASE1_PLAN.md — full text

# Phase 1 — Custody

**Status:** proposal only. No implementation logic written. Nothing below is built yet.

Required by §66.17: custody and authentication changes must be documented and proposed before
they are implemented. Directive 007 stops at this document.

Phase 0 is complete and green at 16 lines. It proved the scarce object: a Wanderer exists in one
place at a time, the service decides where, and a copied client cannot become the real one.

Phase 1 is §67's second block — accounts, passkeys, custody leasing, server-authoritative state,
forced expiration, release, transfer, recovery — under one success condition:

> **The Wanderer must survive hostile or disconnected hosts.**

---

## 1. What Phase 1 proves

One sentence: **custody belongs to an authenticated person rather than to whoever is holding a
string, and no host — however hostile, however disconnected — can keep the Wanderer, retake it,
or destroy it.**

Phase 0 proved a copy is not the real one. Phase 1 proves that the real one cannot be captured.

---

## 2. The two things that change, and why they are the whole phase

### 2.1 A host stops being a string

Today `POST /lease {wanderer: 'W-001', host: 'host-a'}` is the entire admission procedure.
`host` is free text, typed by the caller, believed by the service, and written into the signed
lineage. Anyone can be `host-a`. There is no account, so there is nothing to authenticate, and
§15's first line — `Host authentication → Custody granted` — has no first line.

Phase 1 puts an account underneath the name, and a passkey underneath the account.

### 2.2 A lease stops being a bearer token

This is the more serious of the two, and it is not visible in the Phase 0 test suite because
every attack there is performed by host-a *after* host-a's lease has died.

Today a lease is a signed JSON blob, and `checkLease` asks whether the blob is genuine, known,
live, and current. It never asks **who is presenting it**. So a live lease, if it leaks — copied
off a disk, read out of a log, taken from a compromised client — is custody itself. Whoever holds
the bytes holds the Wanderer, and the service cannot tell them from the host it issued them to.

Phase 1 makes the lease worth nothing on its own. It must be presented *by the account it names*,
proving a live session in the same request. Two independent things must be stolen instead of one,
and the second cannot be stolen by copying, because it rests on a passkey that does not leave the
authenticator.

That change is the reason to do the account work now rather than later, and it should be said
plainly: **Phase 0 shipped a bearer credential.** It was correct for what Phase 0 proved. It is
not correct for a phase whose success condition names hostile hosts.

---

## 3. The chain of custody, end to end

```text
PASSKEY                  a WebAuthn credential on the host's own device.
   │                     The private half never leaves the authenticator,
   │                     so it cannot be copied the way a lease can.
   │  assertion over a server-issued, single-use challenge
   ▼
ACCOUNT                  durable, opaque, one per person. May hold several
   │                     passkeys. Knows no more about the person than it must.
   │  service issues
   ▼
SESSION                  short-lived, absolute timeout, rotated, bound to the
   │                     account and to nothing the client can edit.
   │  used to ASK for custody — never to hold it
   ▼
LEASE                    signed by the Wanderer's own key. Names the Wanderer,
   │                     the ACCOUNT, the EPOCH, a nonce, and an expiry.
   │  presented WITH the session on every call
   ▼
EPOCH                    the question currently being asked. Unchanged from
                         Phase 0, and still the line every attack dies on.
```

The epoch idea from Phase 0 is untouched. Phase 1 adds a gate in front of it and a name
underneath it. Nothing about lineage, signing, or the epoch rule is being redesigned.

### 3.1 What `checkLease` becomes

Phase 0 asks five questions. Phase 1 asks seven, and the two new ones come first and fifth:

```text
0.  settle any lease that has run out          (unchanged — expiry moves the epoch)
1.  is a live session presented?               NEW — no session, no custody
2.  is the lease ours at all?                  (signature)
3.  do we know it?                             (issued, unedited)
4.  has its own life run out?                  (server clock only)
5.  does the session's account own this lease? NEW — the lease is not a bearer token
6.  is it the question being asked now?        (epoch — Phase 0's central line)
7.  has it been given up?                      (still unreachable; still kept)
```

Line 7 has been unreachable since Directive 006, because both ways of letting go now move the
epoch. It stays. Removing a defence because the current ordering makes it redundant is how
orderings come to be depended on silently.

---

## 4. The components

### 4.1 Accounts — `server/src/accounts.js` (new)

An account is the durable identity of a person who may hold Wanderers. It holds as little as
possible (§59: personal information off-chain and minimal by default):

```text
id             opaque, service-generated. Never a name, never an email.
created_at
status         active | suspended | closed
display_name   optional, host-chosen, never written to the lineage
```

**Host numbers are separate.** §34 gives every successful custody period a Journey/Host number
("Host 1,847 of W-001"), and says it must not reveal identity. So the account ID is private to
the service, and what the signed lineage names is the host number for that journey. One account
returning twice is two host numbers, and the public chain cannot tell it was the same person —
which is what §34 and §66.12 ("public verification works without exposing private host
information") together require.

This is a change to what gets signed into an epoch. See §7.2.

### 4.2 Passkeys — `server/src/webauthn.js` (new)

WebAuthn Level 3, per §16. Two ceremonies, both server-driven:

**Registration.** Server issues a single-use challenge, stores it with a short expiry, and on
return verifies the attestation, the challenge, the origin, the RP ID, the flags (user present,
user verified), and stores the credential:

```text
credential_id      unique, indexed
account_id
public_key         COSE, as returned
sign_count
transports
aaguid
backup_eligible / backed_up      (L3: a synced passkey exists on more than one device)
created_at, last_used_at, revoked_at
```

**Authentication.** Server issues a single-use challenge; on return verifies the assertion
signature against the stored public key, the challenge, origin, RP ID, flags, and the sign
counter. A session is issued only if all of it holds.

**On the sign counter.** Many modern authenticators report 0 always, so a counter that fails to
advance cannot be treated as an attack. A counter that goes *backwards* from a non-zero value is
the documented clone signal and will be refused and logged. Both behaviours get a test, because
getting this wrong in either direction is a real failure: refusing legitimate synced passkeys, or
accepting a cloned authenticator.

**Reuse before reinvention (§66.4, §66.15).** I propose `@simplewebauthn/server` rather than
hand-rolling CBOR parsing and attestation verification. Phase 0 deliberately used nothing but
`express`, `better-sqlite3` and Node's own `crypto`, so this is a new dependency and needs
approval — see the open questions. Writing our own WebAuthn verifier is precisely the kind of
security-sensitive reinvention §66.15 warns about, and it would be the least reviewable code in
the project.

### 4.3 Sessions — `server/src/accounts.js`

Short-lived, server-side, and server-expiring — the same discipline §17.1 and §39 demand of
custody, applied to authentication:

```text
absolute timeout        a session dies at a fixed age regardless of activity
idle timeout            and sooner if unused
rotation                the token changes on privilege change (§39: credential rotation)
invalidation            all sessions of an account die on credential revocation
```

A session never *holds* the Wanderer. It only lets an account ask for and use a lease. The two
expire independently and on different clocks, which is deliberate: an expired session with a live
lease must fail, and a live session with an expired lease must fail.

### 4.4 Custody leasing — `server/src/wanderer.js` (modified)

`lease()` stops taking a host string. It takes a session, resolves the account, allocates the
host number for the journey, and issues the lease to the account. Everything downstream — epochs,
signing, lineage — is unchanged.

### 4.5 Server-authoritative state

Already true in Phase 0 and must stay true. Phase 1 adds what §66.10 asks for and Phase 0 has not
yet proved on its own terms:

- **state versions are ordered** — an explicit monotonic `state_version` per Wanderer, alongside
  the existing `seq`;
- **stale states are rejected** — a write may carry a precondition ("I believe the state is at
  version N"), and a write against a version that has moved is refused rather than silently
  applied on top;
- **transitions can be audited** — an append-only audit record for every custody and
  authentication event (§39: immutable/auditable security logs);
- **survives client shutdown and client deletion** — proved by restarting the service against the
  same store and destroying the client's copy entirely.

State *fingerprints*, the Living Mark and the attestation chain are Phase 2 and are not built
here. The existing `state_hash` on each epoch row stays as it is.

### 4.6 Forced expiration

Phase 0 already does the hard part: expiry is server-side, settled from the server's clock by
whoever asks next, recorded at the lease's own expiry time rather than the moment it was noticed,
and it closes the epoch. Nothing about that changes.

Phase 1 adds:

- **operator-forced expiration** — the custody authority (§61) can end a custody period before its
  time. Same code path as expiry: the epoch closes, held by nobody, at the moment of the decision.
  This is the mechanism §17.1 needs against a host who is hostile rather than merely absent.
- **a periodic sweep** — expiry is currently settled lazily, on observation. That is correct and
  keeps history consistent between observers, but it means a Wanderer nobody is watching stays
  formally held until someone looks. A sweep makes the transition happen on time in fact as well
  as in the record. It changes no answer, only when the row is written.

### 4.7 Release

Unchanged in behaviour (Directive 006: letting go closes the epoch, however a host lets go). Phase
1 only requires that the releasing account is the account that holds it, proved by session.

### 4.8 Transfer

Phase 0 performs a transfer as release-then-lease, and that stays the mechanism. Phase 1 makes it
a service-authoritative operation: **custody is assigned by the service, not claimed by whoever
asks first**, and it must complete without any cooperation from the previous holder.

**Who is next remains an open product decision.** §48 lists five candidate selection mechanisms
and settles on none; §65-C leaves it open; §66.18 forbids me deciding it; and Directive 005's Q2
said explicitly that host-selection rules are Phase 1 product decisions to be built later. So
Phase 1 builds the *transfer*, with the *selection* behind a deliberately trivial seam — an
operator naming the next account — and nothing that presumes any of the five mechanisms.

### 4.9 Recovery

Two different things share this word, and the plan keeps them apart.

**Wanderer recovery (§40, §66.10).** A Wanderer must not die because a host loses power, deletes
the app, destroys the computer, or refuses to cooperate. Canonical state already lives only in the
service, so most of §40 is met by Phase 0's architecture. What Phase 1 adds is the explicit
event: restoring from the last valid canonical checkpoint is **a lineage event of its own**, signed
and linked like any other, never a silent rewrite:

```text
W-001
Epoch 3412
RECOVERY EVENT
State restored from verified canonical checkpoint
```

The verifier reports it. Every epoch before it is untouched and still chains. §40's sentence — "a
recovery event should preserve continuity and, if necessary, become another verifiable lineage
event rather than secretly rewriting history" — is the acceptance criterion, and it is testable
exactly as written.

**Account recovery.** A host who loses their only authenticator loses their account, and every
mechanism for giving it back is also a mechanism for taking someone else's. §66.15 lists account
takeover as a named review concern. This is the single most dangerous surface in Phase 1, so the
plan's position is:

- **build:** multiple passkeys per account, enrolled at registration; credential revocation;
  revocation kills live sessions. A host with two devices recovers by using the other one, and
  that path introduces no new authority at all.
- **do not build yet:** any operator-mediated account recovery, email/SMS resets, or recovery
  codes. Each is a takeover path and each is a product decision about how much identity we hold
  on a person — which §59 and §66.18 both say is not mine to make.

See the open questions.

---

## 5. Hostile or disconnected hosts — what must be survived

The success condition of Phase 1, taken as a list. The first block is §66.16's threat list; the
second is new to Phase 1 because it only exists once there are accounts.

| Scenario | What must happen | Why |
| --- | --- | --- |
| Host disconnects the machine and keeps it offline | Lease runs out on the server clock, epoch closes at that moment, Wanderer transfers without them | §17.1 |
| Host alters its system clock | Nothing changes. No client-supplied time is read anywhere | §17.1 |
| Host edits `expires_at` in its own lease and presents it | Refused: the lease is verified against the issued row, not read from the claim | §66.16 |
| Host modifies its local database or state copy | Refused. The service reads its own store and never the host's | §12.1, §66.9 |
| Host clones the whole client | Refused. The clone has no session, and its lease names an account it cannot prove | §66.9 |
| Host replays a custody token | Refused: nonce recorded, epoch moved, session required | §66.9 |
| Host refuses to release | Operator-forced expiration ends it; the epoch closes | §17.1, §61 |
| Host vanishes mid-epoch, service restarts | Custody is recoverable, state intact, lineage unbroken | §66.11 |
| Host attempts an unauthorized Genesis | Refused. Genesis authority is not a host authority | §66.12, §61 |
| Host tries to skip or reorder epochs | Refused, and the verifier detects it in the chain | §66.16 |
| Host presents another Wanderer's lease | Refused: the lease names its Wanderer and is signed by that Wanderer's key | §66.16 |
| **A live lease leaks to a third party** | **Refused without the account's session — the lease alone is not custody** | §39, new |
| A session token leaks | Refused without a lease; and the session dies on absolute timeout regardless of use | §39 |
| A WebAuthn assertion is replayed | Refused: challenges are single-use and expire | §16 |
| An assertion arrives from a different origin or RP ID | Refused | §16 |
| Account B registers a passkey onto account A | Refused: registration is bound to the authenticated account | §66.15 |
| A cloned authenticator is used | Sign counter regression refused and logged | §16 |
| A host account calls an operator route | Refused: custody authority is a separate credential | §61 |
| Account A presents account B's live lease | Refused | new |

The last row and the bolded row are the two that Phase 0 would fail today.

---

## 6. The acceptance tests

Phase 0's sixteen lines stay, unchanged, and must stay green. Phase 1 adds its own suite, run as
one command, every line passing or DENIED with no interpretation needed — the same shape as
`server/src/acceptance.js`.

**Headless WebAuthn.** The ceremonies cannot be driven from a CLI, and a suite that needs a
browser is a suite that will not run in the watcher. So the tests use a software authenticator
written for the purpose (`server/src/test-authenticator.js`) that performs the client half of
WebAuthn with a local key — the same thing the WebAuthn Level 3 virtual authenticator does. It is
test-only and never shipped. The real ceremony needs a browser page; see the open questions.

```text
ACCOUNTS AND PASSKEYS
 1  Account created, passkey registered, attestation verified          ✓
 2  Passkey authenticates, session issued                              ✓
 3  Registration challenge is single-use — replayed                    DENIED
 4  Assertion replayed with the same challenge                         DENIED
 5  Assertion from the wrong origin / RP ID                            DENIED
 6  Assertion signed by a different key for a known credential ID      DENIED
 7  Sign counter goes backwards (cloned authenticator)                 DENIED
 8  Synced passkey reporting counter 0 every time                      ✓ (not an attack)
 9  Account B registers a passkey onto account A                       DENIED
10  Second passkey authenticates after the first is revoked            ✓
11  Revoked credential is used                                         DENIED
12  Revocation kills live sessions                                     DENIED

CUSTODY BOUND TO AN ACCOUNT
13  Lease claimed with a live session                                  ✓
14  Lease claimed with no session                                      DENIED
15  Account A's live lease presented with account B's session          DENIED
16  A live lease presented with no session at all                      DENIED
17  Session hits its absolute timeout, lease still live                DENIED
18  A second account asks while it is held                             DENIED

SERVER-AUTHORITATIVE STATE
19  State survives service restart — epoch, custody, lineage intact    ✓
20  Client's copy deleted entirely; W-001 unaffected                   ✓
21  Write against a stale state version                                DENIED
22  Lease with a client-edited expires_at                              DENIED

FORCED EXPIRATION, RELEASE, TRANSFER
23  Disconnected holder runs out; epoch closes at the expiry moment    ✓
24  Operator forces expiry mid-lease; holder refused immediately       ✓ / DENIED
25  Custody transfers with no cooperation from the previous holder     ✓
26  Previous holder after transfer                                     DENIED
27  A host session calls an operator route                             DENIED
28  A host session attempts Genesis                                    DENIED

RECOVERY
29  Recovery from the last valid canonical checkpoint restores state   ✓
30  Recovery is a signed lineage event; verifier reports it            ✓
31  Recovery rewrites nothing: prior epochs and hashes unchanged       ✓
32  Custody recovered after a host vanished mid-epoch                  ✓

PRIVACY AND LINEAGE
33  Public verifier exposes no account, name, or credential            ✓
34  Lineage names host numbers only; the same account twice is not
    linkable in the public record                                      ✓
```

Success condition, in the spec's words: **the Wanderer survives hostile or disconnected hosts.**
Lines 15, 17, 23, 24, 25 and 32 are the ones that say it.

---

## 7. §66.17 change documentation

### 7.1 Custody and authentication

```text
ORIGINAL STATE
  Custody is granted to a caller-supplied string. There is no account, no
  authentication, and no session. A lease is a bearer credential: checkLease
  verifies that the blob is genuine, issued, live and current, but never who
  is presenting it.

PROPOSED CHANGE
  Accounts, WebAuthn passkeys and server-side sessions. A lease is issued to
  an account and must be presented together with a live session belonging to
  that account. Two new checks in checkLease: a live session (first), and
  account ownership of the lease (fifth).

REASON
  §15 requires host authentication before custody. §16 requires
  passkeys/WebAuthn. §39 requires strong host authentication, credential
  rotation and invalidation after custody ends. §67 Phase 1 requires survival
  of hostile hosts, and a bearer lease does not survive a hostile host who has
  obtained one.

FILES AFFECTED
  server/src/accounts.js          new — accounts, sessions
  server/src/webauthn.js          new — the two ceremonies
  server/src/test-authenticator.js new — test-only client half
  server/src/wanderer.js          lease(), checkLease(), release(), transfer()
  server/src/store.js             accounts, credentials, sessions, challenges,
                                  audit; leases.host → leases.account_id
  server/src/index.js             auth and operator routes
  server/src/config.js            RP ID, origin, session lifetimes
  client/src/index.js             login, session handling
  server/src/acceptance-phase1.js new — the suite above

SECURITY / PRIVACY EFFECT
  A leaked lease stops being sufficient for custody. The strongest remaining
  credential rests on a passkey that cannot be copied off the device. New
  personal data enters the system for the first time — a WebAuthn public key
  and an account row — so §59's minimisation applies from the first commit,
  and the account never reaches the lineage: the public record names host
  numbers only.

TESTS REQUIRED
  Lines 1–18 and 33–34 above. Phase 0's 16 lines must stay green.

RESULT
  Not implemented. Awaiting approval per §66.17 and Directive 007.
```

### 7.2 The lineage record

```text
ORIGINAL STATE
  epochs.host holds the caller-supplied host string, and that string is inside
  the signed epoch body.

PROPOSED CHANGE
  The signed body names an opaque host number rather than a host string. The
  account ID stays in the service's private tables and is never signed and
  never served by the verifier.

REASON
  §34 (host number, must not reveal identity), §66.12 (public verification
  without exposing private host information), §59 (personal data off-chain by
  default). Once hosts are real people this string is personal data, and Phase 2
  puts these records towards a blockchain, where a mistake is not erasable.

FILES AFFECTED
  server/src/store.js, server/src/wanderer.js (openEpoch, verifyLineage)

SECURITY / PRIVACY EFFECT
  The public record becomes unlinkable to a person, and stays that way when
  Phase 2 attests it.

TESTS REQUIRED
  Lines 33–34. Phase 0's verifier lines must still pass with the new body shape.

RESULT
  Not implemented. Note that this changes the shape of what is signed, so
  existing lineages do not verify against the new code. See §8.
```

---

## 8. Migration

§66.6 forbids destructive migration without review, so it is stated rather than assumed:
**there is no real data.** Everything in `data/` is a test artifact, the acceptance suites delete
and re-mint their databases on every run, and no Wanderer has ever been held by a person. Phase 1
therefore proposes a fresh schema and no migration path, and `data/cli.db` — the only store that
persists between runs — would be re-minted. If any of that is wrong, it needs saying before
implementation begins, because the lineage body change in §7.2 cannot be applied to an existing
chain without breaking its signatures.

---

## 9. Built with

Node 22, as Phase 0. `express`, `better-sqlite3`, Node's own `crypto` for Ed25519 signing and
hashing — all unchanged.

**One new dependency proposed:** `@simplewebauthn/server` for the WebAuthn ceremonies. See §4.2
and the open questions.

---

## 10. Deliberately out of scope for Phase 1

The Genesis registry, public verification beyond the existing verifier, the attestation chain,
state fingerprints, the Living Mark and the lineage viewer (all Phase 2). Semantic memory and
privacy classification (Phase 3). The Wisp, voice and senses (Phase 4). The Passport, Journey
records, Host Certificates and Verifiable Credentials (Phase 5). Evolution (Phase 6).

Also out of scope, and each for a reason rather than by omission:

- **Host selection** — §48, §65-C, and Directive 005 Q2. The transfer is built; who is chosen is not.
- **Visit duration** — §19.1 and §65-B are open. `LEASE_SECONDS` stays a test value with no product meaning.
- **Re-lease restrictions and cooling periods** — Directive 005 Q2 said build nothing for them, and nothing here does.
- **Operator-mediated account recovery** — §4.9.
- **Rate limiting, anomaly detection, encrypted transport and encrypted storage** — §39 requires
  all four in production. TLS and rate limiting belong to deployment rather than to this
  architecture, and there is no deployment yet. Named here so they are not later assumed done.
- **Independent security review** — §66.15 requires it before real users, and this plan does not
  substitute for it.

---

## 11. Open questions for Lonnie

Product decisions, per §66.18, and one dependency question.

1. **Account recovery.** A host with one passkey and a lost device is locked out permanently.
   The plan builds only multi-passkey enrolment, because every other route back in is also a
   route in for someone else. Options: require two passkeys at registration; operator-mediated
   recovery with an audited log; recovery codes shown once; or accept lockout. Which?

2. **How much do we know about a host?** An account can be genuinely anonymous — a passkey and
   nothing else, no email, no name. That is the strongest privacy position and it makes recovery
   and abuse-handling much harder. How anonymous should a host be allowed to be?

3. **`@simplewebauthn/server`.** Phase 0 was built on express, better-sqlite3 and Node crypto
   alone. Approve the dependency, or require a hand-rolled verifier? My recommendation is
   strongly the former, per §66.4 and §66.15.

4. **The real login surface.** WebAuthn needs a browser; the client is a CLI. Options: a minimal
   local page for the ceremony that hands a session back to the CLI; or defer real login to the
   Phase 4 client and run Phase 1 on the test authenticator alone. The acceptance suite is
   headless either way.

5. **RP ID and origin.** `localhost` for development is assumed. Is there a production domain to
   design against now, or is that a Phase 7 question?

6. **Who is the operator?** Phase 1 needs a custody authority (§61) to force expiry and assign
   transfers. Is that a real credential with its own account type in Phase 1, or a service-local
   administrative call with no remote surface until there is somewhere to deploy it?

7. **Host numbers.** Confirmed as per-journey and per-Wanderer ("Host 1,847 of W-001"), allocated
   at lease time and never reused? And confirmed that the same account returning receives a new
   number rather than its old one — which is what makes the public record unlinkable, but does
   mean the public chain cannot show that a Wanderer returned to a previous host.

8. **One Wanderer or several.** The store has held many since Phase 0 and Phase 1 needs only
   W-001. Should an account be able to hold two Wanderers at once, or is one-at-a-time a rule
   worth building in now?
