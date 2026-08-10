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

---

## Directive 008 — Phase 1 implemented: Custody — DONE

**Commits:** `e488beb` (Phase 1), `c9c7936` (backup-flag correction) in CC-Wanderer.
Tree was clean at `16c6c07` before any change, so the commit-before-change rule was satisfied
with nothing to lose.

**Suite: 50 lines, 50 passed, 0 failed.** Phase 0's sixteen stayed green alongside Phase 1's
thirty-four, as the directive required. One command: `npm run accept`.

---

### What was built

Phase 1's whole argument is two sentences.

**A host stops being a string.** `POST /lease {host: 'host-a'}` was the entire admission
procedure — free text, typed by the caller, believed by the service. §15's first line, *Host
authentication → Custody granted*, had no first line. There is now an account underneath the
name, a passkey underneath the account, and a session proving the account is present.

**A lease stops being a bearer token.** This is the more serious of the two and it was not
visible in Phase 0's suite, because every attack there was performed by host-a *after* host-a's
lease had died. `checkLease` asked whether a lease was genuine, known, live and current. It never
asked **who was presenting it**. So a live lease, if it leaked — copied off a disk, read out of a
log, taken from a compromised client — *was* custody. Phase 0 shipped a bearer credential. It was
correct for what Phase 0 proved and it is not correct for a phase whose success condition names
hostile hosts. A lease now names the account it was issued to and must be presented **with** a
live session belonging to that account. Two independent things must be stolen instead of one, and
the second cannot be stolen by copying.

The epoch idea is untouched. Phase 1 put a gate in front of it and a name underneath it.

`checkLease` went from five questions to seven; the two new ones are first and fifth:

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

**What the lineage names also changed.** `epochs.host` held a caller-supplied string and that
string was inside the signature. It is now §34's host number — an integer, per Wanderer, allocated
at lease, never reused (Directive 008 §7). The account stays in private tables, is never signed
and never served. That mattered now rather than in Phase 2 because Phase 2 puts these records
towards a blockchain, where a mistake is not erasable.

**Files.** New: `accounts.js` (accounts, sessions, audit), `webauthn.js` (the two ceremonies),
`test-authenticator.js` (the client half, test-only), `acceptance-phase1.js` (the suite).
Modified: `store.js`, `wanderer.js`, `index.js`, `config.js`, `client/src/index.js`,
`acceptance.js`.

### The eight answers, as implemented

1. **Recovery** — multi-passkey enrolment only. Lines 10–12 prove enrol-second, revoke-first,
   authenticate-with-second, and that revocation kills live sessions. No other route back in exists.
2. **Anonymity** — passkey and nothing else. An account is an opaque id and a status. The
   `display_name` column exists per the approved plan and is **never written to** by any Phase 1
   code path; the WebAuthn user handle is the opaque account id, so no name reaches the
   authenticator either.
3. **`@simplewebauthn/server`** — installed, v13.3.2. Ours is the policy around it: challenge
   lifetime and single use, account binding, sign-counter meaning, when a session is issued.
4. **Login surface** — the software test authenticator, suite headless, browser ceremony deferred
   to Phase 4.
5. **RP ID / origin** — `localhost`, in `config.js`, per-service overridable.
6. **Operator** — service-local method calls with **no HTTP routes at all**. Line 27 does not test
   that operator routes refuse a host; it tests that there are none to find.
7. **Host numbers** — per-Wanderer, allocated at lease, never reused. Line 34 shows account A
   appearing as host 1 and host 2 with nothing in the public record linking them.
8. **No limit on Wanderers per account** — nothing was built that imposes one.


### Full test output

```text
> node server/src/acceptance.js && node server/src/acceptance-phase1.js


PHASE 0 — PROVE THE SCARCE OBJECT

   1  Genesis: W-001 minted, first record signed           ✓ 
   2  Lease A: host-a holds it                             ✓ epoch 1, host 1
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
  13  Verifier: authentic, current holder, lineage from Genesis ✓ holder host 2, 4 links

      host-b sees 1 line(s) carried forward from host-a

      host-c holds it for 3s (until 2026-08-10T22:24:39.876Z) — epoch 1, held by host 1 — waiting it out
  14  Expiry alone closes the epoch: a verifier sees it held by nobody ✓ epoch 1 → 2, held by null
  15  Expired holder is refused without releasing          DENIED  (lease has expired)
  16  Custody moves on with no release from the expired holder ✓ epoch 2 → 3
      host-d sees 1 line(s) carried forward from host-c

  16 passed, 0 failed


PHASE 1 — CUSTODY BELONGS TO A PERSON, AND A LEASE IS NOT A BEARER TOKEN

      W-001 minted by Genesis authority — epoch 0

  ACCOUNTS AND PASSKEYS

   1  Account created, passkey registered, attestation verified ✓ account acct_58eab7bd…
   2  Passkey authenticates, session issued                    ✓ session expires 2026-08-10T22:39:41.058Z
   3  Registration challenge is single-use — replayed          DENIED  (challenge has already been used)
   4  Assertion replayed with the same challenge               DENIED  (challenge has already been used)
   5  Assertion from the wrong origin / RP ID                  DENIED  (assertion was not verified: Unexpected authentication response origin "http://evil.example", expected "http://localhost"; assertion was not verified: Unexpected RP ID hash)
   6  Assertion signed by a different key for a known credential ID DENIED  (assertion was not verified)
   7  Sign counter goes backwards (cloned authenticator)       DENIED  (assertion was not verified: Response counter value 1 was lower than expected 2)
   8  Synced passkey reporting counter 0 every time            ✓ (not an attack — most passkeys never advance a counter)
   9  Account B registers a passkey onto account A             DENIED  (a passkey can only be enrolled onto the authenticated account)
  10  Second passkey authenticates after the first is revoked  ✓ 1 live session(s) ended with the passkey
  11  Revoked credential is used                               DENIED  (that passkey has been revoked)
  12  Revocation kills live sessions                           DENIED  (session was revoked)

  CUSTODY BOUND TO AN ACCOUNT

  13  Lease claimed with a live session                        ✓ epoch 1, host 1
      the session that asked for custody was rotated as it was granted
  14  Lease claimed with no session                            DENIED  (no session presented)
  15  Account A's live lease presented with account B's session DENIED  (that lease was issued to a different account)
  16  A live lease presented with no session at all            DENIED  (no session presented)
  17  Session hits its absolute timeout, lease still live      DENIED  (session reached its absolute timeout)
      the lease itself had not expired — it runs until 2026-08-10T22:25:41.173Z
  18  A second account asks while it is held                   DENIED  (W-001 is held by host 1 until 2026-08-10T22:25:41.135Z)

  SERVER-AUTHORITATIVE STATE

  19  State survives service restart — epoch, custody, lineage intact ✓ epoch 1, host 1, 2 links
  20  Client's copy deleted entirely; W-001 unaffected         ✓ 1 line(s) still canonical with nothing left on the host
  21  Write against a stale state version                      DENIED  (state has moved on: you wrote against version 0, it is at 1)
      state is at version 1; the write claimed version 0
  22  Lease with a client-edited expires_at                    DENIED  (lease is not signed by this Wanderer)

  FORCED EXPIRATION, RELEASE, TRANSFER

      host 1 holds it for 3s (until 2026-08-10T22:24:46.670Z) — then disconnects and is never heard from again
  23  Disconnected holder runs out; epoch closes at the expiry moment ✓ epoch 1 → 2, closed at 2026-08-10T22:24:46.670Z — the lease's own moment, not ours
  24  Operator forces expiry mid-lease; holder refused immediately ✓ epoch 1 → 2; holder: lease is for epoch 1; the current epoch is 2
      account A re-leased with no cooling period — it is host 2 now, and was host 1 before
  25  Custody transfers with no cooperation from the previous holder ✓ host 2 → host 3, epoch 5, 1 line(s) carried forward
  26  Previous holder after transfer                           DENIED  (lease is for epoch 3; the current epoch is 5)
  27  A host session calls an operator route                   DENIED  (404; 404; 404)
  28  A host session attempts Genesis                          DENIED  (genesis requires Genesis authority; a host credential is not one; genesis requires Genesis authority; a host credential is not one)

  RECOVERY

  29  Recovery from the last valid canonical checkpoint restores state ✓ 1 line(s) lost, 1 restored from checkpoint at version 1
  30  Recovery is a signed lineage event; verifier reports it  ✓ W-001 / epoch 6 / RECOVERY EVENT — chain still verifies from Genesis
  31  Recovery rewrites nothing: prior epochs and hashes unchanged ✓ 6 prior link(s) signed-identical; the store refuses a rewrite outright
  32  Custody recovered after a host vanished mid-epoch        ✓ host 2 holds it; the vanished host 1's line is still there

  PRIVACY AND LINEAGE

  33  Public verifier exposes no account, name, or credential  ✓ 9 private identifiers checked against 2775 bytes of public record
  34  Lineage names host numbers only; the same account twice is not
      linkable in the public record ✓ account A appears as host 1 and host 2, and as nothing else

  34 passed, 0 failed

```

---

### Deviations

Ten, and none of them silent.

**1. Phase 0's suite could not stay literally unchanged, and did not.**
PHASE1_PLAN.md §6 said "Phase 0's sixteen lines stay, unchanged". That was not achievable and
should not have been written: the plan itself removes the host string that Phase 0's file typed
in, and changes what the verifier reports. **The sixteen assertions are the same sixteen, in the
same order, in the same words, proving the same things.** What changed is the mechanics —
`host-a` enrols a passkey and logs in, and reads as `host 1` where the lineage is concerned.

The attacks got **harder**, which is the part worth reading. In Phase 0, host A came back with
nothing but a dead lease. It now comes back with a dead lease *and a perfectly live session* — it
is still logged in, still authenticated impeccably. So none of the five attacks can die for want
of a credential. Each still dies on the line Phase 0 said it would: the epoch it names is not the
question being asked.

**2. A real bug, found by line 15 and fixed.**
The first implementation returned the lease as `{ok: true, ...lease, session}`. A host handing
that object straight back presented `ok` and `session` as though they were signed fields, so
**every genuine lease failed its own signature check** — and failed it *identically to a forgery*.
The suite caught it because line 15 reported "lease is not signed by this Wanderer" where it
should have reported "issued to a different account". Lines 15, 19, 20, 21 and 22 were all
passing or crashing for the wrong reason. The lease is now returned as its own object,
`{ok, lease, session}`, and what is signed and what is presented are the same bytes.

**3. `/verify` now returns the lineage array.** Plan §10 puts "public verification beyond the
existing verifier" in Phase 2. Lines 23, 30, 31, 33 and 34 cannot be proved without it — a claim
that the public record exposes nothing needs the public record. Every field returned is already
inside a signature and none of it names anybody. Flagged because it is a step into Phase 2's
territory, small and deliberate.

**4. Sessions are NOT revoked when custody ends.** §39 lists "invalidation after custody ends",
and the first implementation did it. It is wrong twice: Directive 005 Q2 requires unrestricted
lease/release/re-lease by the same host, and logging someone out for handing something back
breaks that; and it would have weakened Phase 0's attacks to nothing by refusing them at the
session line before they ever reached the epoch. **The custody credential is the lease**, and it
is dead the instant the epoch moves — that is the invalidation §39 asks for. The session is
authentication, not custody, and has its own two timeouts. Line 24 of the suite is the demonstration.

**5. Genesis authority was added.** Line 28 (§66.12) requires that a host cannot mint a Wanderer,
and Phase 0's `/genesis` was open to anyone. It is now guarded by a service-local secret, held by
whoever stands up the service, generated randomly if not supplied — so an unattended service can
mint nothing. The plan's threat table named this; its component list did not.

**6. The store refuses an old schema rather than migrating it.** Plan §8 said `data/cli.db` would
be re-minted. Implemented as a refusal with an explicit message rather than an automatic drop,
because §66.6 forbids destructive migration without review. Verified: the old Phase 0 store is
refused by name and version.

**7. Append-only is now enforced by the database, not just respected by the code.** Triggers
refuse to delete an epoch, to rewrite any signed field of one, or to touch the audit log at all.
`closed_at` is the single field an epoch may gain. Beyond the plan's letter; §66.10 asks that
transitions be auditable, and an audit trail the application layer may quietly rewrite is not one.
Line 31 proves the refusal rather than assuming it.

**8. The test authenticator's private key is exportable.** A real authenticator's is not — that is
the entire point of §16. The CLI has nowhere to keep a key between invocations, so `enrol`/`login`
store it in the host's own file and **say so on every use**. The server half is real; this half is
a stand-in, and the property returns with the browser ceremony in Phase 4 (Directive 008 §4). It
does not affect the suite, which drives the ceremonies in-process.

**9. The periodic sweep is built but has no numbered acceptance line.** The plan's 34 lines do not
include one. It is implemented (`startSweep`), wired into the standalone service, and changes no
answer — only when a row gets written. Lazy settling on observation is still what the suite proves.

**10. Line 6 needed its counter raised to prove anything.** "Assertion signed by a different key
for a known credential ID" was initially refused for counter regression rather than for the
signature — the right answer for the wrong reason. The impostor now presents a counter above the
stored one, so the signature is what fails.

### Still not done, and named so it is not assumed done

- **Independent security review (§66.15).** Not a deviation, a standing requirement. Nothing here
  substitutes for it, and it is required before real users.
- Rate limiting, anomaly detection, encrypted transport, encrypted storage (§39) — deployment
  concerns, and there is no deployment.
- Host selection (§48, §65-C) — the transfer is built; who is chosen is not, and nothing here
  presumes any of the five mechanisms. The operator names the account, and that seam is deliberately trivial.
- Operator-mediated account recovery, recovery codes, email/SMS reset — each a takeover path, each
  a product decision (Directive 008 §1).
- Visit duration (§19.1, §65-B) — `LEASE_SECONDS` remains a test value with no product meaning.

### Open question

Only one, and it is not blocking.

**Should the operator have a credential yet?** Directive 008 §6 settled that the custody authority
is a service-local call with no remote surface, and that is what was built — but it means the
authority is currently *anyone with code running in the process*, which is the correct answer for
a service with nowhere to deploy and the wrong one the moment there is somewhere. It does not need
answering until there is. Raised now so it is not discovered later.

Phase 1's success condition, in the spec's words: **the Wanderer must survive hostile or
disconnected hosts.** Lines 15, 17, 23, 24, 25 and 32 are the ones that say it. Line 15 — one
account's live lease presented with another account's session — is the line Phase 0 would have
failed.

Awaiting direction on Phase 2.

---

# Directive 009 — Watcher push fix, and the Phase 2 plan — PART A DONE, PART B AWAITING APPROVAL

Both parts done as written. Part A is fixed and tested. Part B is a proposal and stops for
approval, per §66.17. No Phase 2 implementation exists.

Baseline before either part: the suite ran green at **50 (16 + 34), 0 failed, exit 0**, unchanged.

---

## Part A — the silent failure, and what it actually was

### It was not a failed push. There was no push.

The directive describes the Phase 1 report as having "committed locally but the push to the relay
failed silently (status 0, nothing landed)." The first half is right and the diagnosis is not, which
matters because a retry does not fix it.

The watcher keeps `claude -p` output in `watcher.log`, but that is only the final message — the tool
calls are in the session transcript. The Directive 008 run is
`~/.claude/projects/-home-nobara-user-wanderer-relay/04d426da-….jsonl`. Every git command it ran:

```text
git log --oneline | head -20 …            (looking around)
git status --porcelain … TREE CLEAN AT …  (commit-before-change check)
git add -A && git status --short
git commit -q -F - <<'EOF' Phase 1: custody belongs to a person …
git add -A && git commit -q -m "record backup eligibility …"
git add REPORTS.md && git commit -q -m "Directive 008: Phase 1 built, suite green at 50 …"
git status --short && echo "CC-Wanderer: …" && echo "relay: …"
```

No `git push`. The string does not occur anywhere in that transcript — 0 occurrences, against
exactly 1 in each of the four earlier cycles that did land. The run committed, checked that both
trees were clean, correctly reported *"Both repos clean and committed"*, and exited 0 having never
attempted delivery. It was not lying. It was answering a narrower question than the one that
mattered.

So the failure mode is worse than a broken push, and more ordinary: **the last step was simply
omitted, and nothing anywhere was responsible for noticing.** `b770543` reached origin later by a
push from this repository that no Claude session made — someone pushed it by hand.

That changes the fix. Retry logic inside the directive's own instructions cannot help, because the
run believed it had finished. The check has to live outside the run, has to ask the remote rather
than the run, and must not trust the run's exit status.

### What changed in watch.sh

Dated backup kept first, as instructed:

```text
/home/nobara-user/relay-watcher/watch.sh.2026-08-10-before-push-check
```

A `verify_push()` function now runs after every directive, before the state file is written. It:

1. **Warns if the run left anything uncommitted** — a report written and never committed is the
   same loss by another route. It does not commit on the run's behalf; a human decides what stray
   files are. It does not fail delivery either (committed work can still land, and a scratch file
   must not raise a false alarm every minute), but it does stop the cycle calling itself clean.
2. **Fetches origin and compares** `origin/master...HEAD`. In sync → one quiet line and done.
3. **Pushes if there are local commits origin lacks**, then **re-fetches and compares again** rather
   than believing the push's exit status — the whole point of the exercise is that a step reporting
   success is not evidence it happened.
4. **Retries once**, then gives up loudly.

Loud means a bannered `ALARM` block, greppable on one word, naming the undelivered commits and the
command to fix it by hand:

```text
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
2026-08-10T15:38:51-07:00  ALARM  RELAY PUSH FAILED -- 1 local commit(s) are NOT on origin after 2 attempts
                                  undelivered: 5156708 Directive report 3 - will not land
                                  the work is committed locally and safe; the relay has not been told
                                  fix by hand: git -C /…/relay push origin master
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

The final line of every cycle is now a verdict rather than a status code:

```text
cycle complete -- directive run and report delivered
cycle complete WITH WARNINGS -- report delivered, but see ALARM above
cycle INCOMPLETE -- run status 0, delivery status 1 (see ALARM above)
```

and the script exits non-zero when either half failed, so `systemctl --user status
relay-watcher.service` shows a failed unit instead of a clean one. The timer keeps firing; only the
last run is marked.

### It was tested, not just written

`bash -n` clean. Then a scratch bare repo and clone, with the real function text extracted from
`watch.sh` so the test exercised shipped code rather than a copy, across five paths:

| path                                       | result |
|--------------------------------------------|--------|
| already in sync                            | one quiet line, returns 0 |
| local commits ahead, push works            | pushes, re-fetches, confirms landed, returns 0 |
| push impossible (unwritable remote)        | 2 attempts, then ALARM naming the commit, returns 1 |
| run left REPORTS.md uncommitted            | ALARM, delivery still verified, cycle marked WITH WARNINGS |
| origin unreachable entirely                | ALARM distinguishing "cannot check" from "did not land", returns 1 |

**One thing Part A does not fix.** It catches an undelivered report; it cannot make a run write a
report it never wrote. If a future directive produces nothing, the watcher will correctly say the
relay is in sync and be right. The gap between "delivered" and "worth delivering" is still read by
a person.

**This cycle does not exercise it, and that is correct.** A directive that amends the watcher is
being executed *by* the watcher, so the running shell had already opened the old `watch.sh` before
the new one existed — `/proc/776572/fd/255` points at an unlinked inode holding the original bytes.
It will therefore finish this cycle on the old logic, and the new check runs for the first time on
the next directive. This report's own delivery was verified by hand instead, three ways: a re-fetch
and comparison after the push, `git rev-parse` on both sides, and `git ls-remote` asking GitHub
directly. All three agree on `2e3329b`.

That behaviour is also a hazard worth naming, because it was luck rather than design. **bash reads a
script lazily by byte offset.** Writing to a running script *in place* moves the ground under the
shell's own resume point: it carries on reading at the old offset and executes whatever fragment
now lives there. My edits inserted about ninety lines *above* the currently-executing line, which is
exactly the case that breaks. It was safe only because the editing tool writes a new file and
renames it, leaving the running shell holding the old inode. Relying on that is not acceptable for
the one script that edits itself every time a directive says so, so `watch.sh` now carries the rule
at the top: **amend it via a temp file and `mv`, never in place.** Verified after the swap that the
running instance still holds the original bytes and the next run gets the new file.

---

## Part B — PHASE2_PLAN.md

Committed to CC-Wanderer as `a975413`. Full text below, as Directive 009 asks.

Three things worth reading before the plan itself, because they are the parts that changed my mind
while writing it:

**1. The verifier does not check the key.** I set up two services, each with its own Genesis
authority, each minting its own `W-001`, and asked each to verify itself. Both said
`authentic=true`, `problems=[]`. Different keys, same verdict, and nothing in the output naming
which one answered — because `verifyLineage` reads the public key out of the same database that
produced the chain it is checking. Phase 2's real subject is therefore not the blockchain; it is
moving W-001's identity key somewhere the service cannot rewrite. The blockchain is how, not what.

**2. `state_hash` is a commitment over the host's own text, and it was about to be published.**
It is `sha256` over the state rows, and those rows contain `line` — what the host wrote. Private,
that is useful. On an immutable public ledger it is a permanent confirmation oracle for guessable
private content: anyone with a candidate line can check it and never be un-checkable again. §41 asks
for a manifest of counters precisely to avoid this, and §10 forbids conversation content on-chain —
a commitment to content being a form of it. The plan splits it and line 24 of the suite exists to
keep it split for good.

**3. What I have not claimed.** A clone holding a verbatim copy of epochs 0…N is, for those epochs,
byte-identical to the real thing, and no registry changes that because nothing is wrong with those
bytes. What it cannot do is extend the chain. So the distinguishing fact is never "the past looks
wrong", it is "the head is here and it is not you" — §13.2's fork. That is stated as its own
acceptance line rather than left for the phase to imply something stronger.

The plan proposes 31 acceptance lines, 81 with Phase 0 and Phase 1. **Line 7 is the success
condition and it fails today.** There are eight open questions and three of them change what gets
built — the testnet and the local-ledger split, whether every journey is its own transaction, and
whether the public lineage lists journey numbers at all.

---

# Phase 2 — Authenticity

**Status:** proposal only. No implementation logic written. Nothing below is built yet.

Required by §66.17: changes involving blockchain and signing must be documented and proposed
before they are implemented. Directive 009 stops at this document.

Phase 0 is green at 16 lines and Phase 1 at 34, fifty in total. Together they proved two things:
a Wanderer exists in one place at a time and a copied client cannot become the real one (Phase 0),
and custody belongs to an authenticated person rather than to whoever holds a string, so no host
can keep, retake or destroy the Wanderer (Phase 1).

Phase 2 is §67's third block — Genesis registry, public verification, attestation chain, state
fingerprints, Living Mark, lineage viewer — under one success condition:

> **Prove that W-001 can be independently distinguished from a clone.**

---

## 1. What Phase 2 proves, and the word that carries all the weight

The success condition contains one word that Phases 0 and 1 never had to satisfy:
**independently.**

Phase 0's verifier already distinguishes the real W-001 from a copied *client*. It does it by
asking the service, and the service knows. Phase 2 has to distinguish the real W-001 from a copied
*service* — and it has to do it for somebody who does not trust us, cannot ask us, and would get
the same confident answer from the counterfeit if they did.

### 1.1 The gap, demonstrated rather than asserted

`verifyLineage` re-walks every link from Genesis and re-checks every signature. It is honest work.
But look at where it gets the key it checks against (`server/src/wanderer.js:487,496`):

```js
const w = this.wanderer(id)                                    // ...from our own database
if (!verify(body, row.signature, w.public_key)) problems.push(...)
```

The chain is verified against a public key read out of the same database that produced the chain.
That is a closed loop, and a counterfeit gets to close it too. Two services were stood up, each
with its own Genesis authority, each minting its own `W-001`, and each asked to verify itself:

```text
REAL   authentic=true  epoch=0  lineage_verified=true  problems=[]
CLONE  authentic=true  epoch=0  lineage_verified=true  problems=[]

real  W-001 public key : MCowBQYDK2VwAyEAzq7BSDRGBsu97xnBWYJQy+wt...
clone W-001 public key : MCowBQYDK2VwAyEAwIZbXwonZcIM8Fw4fpwE3Z/y...
```

Two different keys, two different Wanderers, the same verdict, and nothing in the verifier's output
naming which of them answered. The keys differ and **nobody is checking the key**.

So Phase 2's real subject is not the blockchain. It is this:

> **W-001's identity key must be established somewhere the service cannot rewrite, before the
> service is asked to prove anything with it.**

That is what §5.1's "Genesis attestation" is for, what §7.1 means by "birth registry", and why
§66.12 lists *"a copied Wanderer cannot produce valid current provenance"* as a separate criterion
from *"each authenticated transition references its predecessor"*. The chain-walking half is built.
The anchor is not.

### 1.2 What we will not claim to have proved

Honesty about the limit, stated now rather than discovered in the report.

A clone that copies the real chain **verbatim** up to epoch N is, for epochs 0…N, byte-identical to
the real thing. No registry fixes that, because there is nothing wrong with those bytes. What the
clone cannot do is **extend** the chain: epoch N+1 needs a signature from a private key it does not
have, and the registry says which key that must be.

So the distinguishing fact is never "the past looks wrong." It is **"the head is here, and it is
not you."** This is §13.1 and §13.2 exactly — *a copy may resemble a Wanderer, but it cannot become
the canonical Wanderer* — and the acceptance suite will state it as its own line (line 10) rather
than let the phase quietly imply something stronger.

---

## 2. How EAS attestations map onto the epoch chain we already have

### 2.1 What exists

An epoch row is signed over this body, and the chain is the `prev_hash` field
(`server/src/wanderer.js:186`, `495`):

```text
{ wanderer, epoch, event, host_number, opened_at, prev_hash, state_hash }
signature = Ed25519(body, wanderer private key)
prev_hash = sha256(previous row's signature)      -- 'GENESIS' for epoch 0
```

Append-only, and enforced by the database rather than by our good intentions: triggers refuse to
delete an epoch or to rewrite any signed field, and `closed_at` is the only field a row may gain
(`server/src/store.js:191-214`).

### 2.2 What §9 asks the public record to contain

```text
wandererID  epochNumber  eventType  stateManifestHash
previousAttestationUID  timestamp  protocolVersion  issuer  signature
```

### 2.3 The mapping

One EAS attestation per epoch row. The correspondence is close to exact, which is the good news —
Phase 0 built the right shape without knowing it:

| §9 attestation field     | where it comes from                                        |
|--------------------------|------------------------------------------------------------|
| `wandererID`             | `epochs.wanderer_id`                                       |
| `epochNumber`            | `epochs.epoch`                                             |
| `eventType`              | `epochs.event`, widened — see §2.5                         |
| `stateManifestHash`      | **new** — the §41 manifest, *not* today's `state_hash` (§5) |
| `previousAttestationUID` | EAS `refUID` → the previous epoch's attestation            |
| `timestamp`              | `epochs.opened_at`                                         |
| `protocolVersion`        | **new** — a constant, `1`, recorded so it can change       |
| `issuer`                 | the attester address; for Genesis, the Genesis authority   |
| `signature`              | `epochs.signature` — the Wanderer's own Ed25519 signature  |

### 2.4 Two chains, pinned to each other in both directions

This is the part that has to be right, so it gets stated plainly.

After Phase 2 there are two chains over the same events: the local one (`prev_hash` over Ed25519
signatures) and the public one (`refUID` over attestation UIDs). If they are merely parallel, an
attacker picks whichever is more convenient. They must be **pinned**:

```text
   LOCAL CHAIN                              PUBLIC CHAIN
   epoch N-1  ──prev_hash──┐                attestation N-1 ──refUID──┐
                           │                                          │
   epoch N ────────────────┘                attestation N ────────────┘
      signature  ──────────────commits to───────►  epoch_link
      (Ed25519 over the body)                      = sha256(signature)
                 ◄─────────────records────────      attestation UID
```

- **Public → local.** The attestation carries `epoch_link = sha256(epochs.signature)` — the same
  value the *next* local row uses as its `prev_hash`. So the on-chain record commits to the exact
  local link. Altering a local epoch breaks agreement with a record we cannot edit.
- **Local → public.** The service records which attestation belongs to which epoch, so a verifier
  can walk from either end and find the other.

Either direction alone leaves a hole. Public→local alone cannot detect an epoch we simply never
attested (a silent gap); local→public alone can be edited by whoever edits the database. Both
directions, and the suite tests **both gaps** — a local epoch with no attestation (line 16) and an
attestation with no local epoch (line 17).

### 2.5 Event types

§9 proposes `GENESIS CUSTODY_BEGIN CUSTODY_END STATE_COMMIT RECOVERY RETIREMENT`. Today
`epochs.event` holds `genesis | custody | recovery`, because Directives 005 and 006 collapsed
custody-end into "a new epoch held by nobody" — an epoch whose `host_number` is `NULL`.

That decision was right and Phase 2 keeps it, so the mapping is:

```text
epoch 0, event=genesis                       → GENESIS
event=custody, host_number = N               → CUSTODY_BEGIN
event=custody, host_number = NULL            → CUSTODY_END
event=recovery                               → RECOVERY
```

`STATE_COMMIT` and `RETIREMENT` are **not** emitted in Phase 2. There is no state-commit event in
the model — state accumulates inside an epoch and is fingerprinted at the epoch boundary — and
nothing in §67 asks for retirement yet. Adding either is an open question (§11.6), not a decision
to make here.

### 2.6 Where the attestation UID is recorded, and why not in `epochs`

The obvious move is a column: `epochs.attestation_uid`. It is the wrong move.

`epochs` is append-only *because a trigger says so*, and the trigger permits exactly one field to
change after the row is written. Adding a second mutable field to the lineage table to hold a value
that arrives later means loosening the guard that Phase 1 deliberately tightened — and loosening it
on the table whose immutability is the entire argument.

So attestations go in their own table, which permits **no** updates at all:

```sql
CREATE TABLE attestations (
  wanderer_id   TEXT NOT NULL,
  epoch         INTEGER NOT NULL,
  uid           TEXT NOT NULL,      -- the EAS attestation UID
  ref_uid       TEXT,               -- the previous epoch's UID; NULL at Genesis
  epoch_link    TEXT NOT NULL,      -- sha256(epochs.signature): the pin to the local chain
  manifest_hash TEXT NOT NULL,
  chain         TEXT NOT NULL,      -- which chain/ledger it lives on
  attester      TEXT NOT NULL,
  placed_at     TEXT NOT NULL,
  onchain       INTEGER NOT NULL,   -- 1 = a transaction; 0 = an offchain EAS attestation
  PRIMARY KEY (wanderer_id, epoch, uid)
);
-- triggers: no delete, no update. Not one field. An attestation is a fact about a public
-- record; if it is wrong, the correction is another row, not an edit.
```

`epochs` and its triggers are untouched. That matters more than the convenience.

---

## 3. What runs on-chain, and what does not

### 3.1 The three tiers

```text
ON-CHAIN, ALWAYS                    Genesis attestations. One per Wanderer, ever.
(a transaction, permanent)          The registry. The root of trust. Rare and worth paying for.

ON-CHAIN, PERIODIC                  Anchors. One attestation whose payload is a commitment over
(a transaction, batched)            a run of epochs, so a long life does not cost per breath.

OFF-CHAIN, EVERY EPOCH              EAS offchain attestations: signed, structured, free, and
(signed, not a transaction)         timestamped on-chain in batches by the anchor above.

NEVER                               §10's list. See §3.4.
```

### 3.2 Why not one transaction per epoch

§8's diagram shows `GENESIS → JOURNEY 0001 → 0002 → 0003`, each referencing the last, and the
straightforward reading is one on-chain attestation per journey. It is the right *shape* and the
wrong *default*, for two reasons that are not cost:

1. **It puts a blockchain in the custody path.** A lease that cannot be issued until a transaction
   confirms is a lease that fails when an RPC endpoint does. Phase 1's whole argument is that the
   service is authoritative and survives things going wrong; making custody wait on a third party
   gives that away. Attestation must be *asynchronous to custody* — the epoch is real when the
   service signs it, and the public record catches up.
2. **It is unbounded.** §33's example shows epoch 9215. Nothing about the design should get
   linearly more expensive as a Wanderer lives longer, because the Wanderer living longer is the
   entire product.

So: every epoch is attested (off-chain, immediately, free), and the off-chain attestations are
anchored on-chain in batches. Both are EAS; the difference is whether a transaction is sent.

**The honest cost of that choice:** an off-chain EAS attestation is a signed statement, not a
timestamped one. Until it is anchored, its *ordering* rests on our signature rather than on a
block — we could in principle sign a different epoch N later and discard the first. Anchoring is
what removes that, so the anchor interval is exactly the window in which we are trusted, and it
should be stated publicly rather than buried. §11.2 asks Lonnie whether that window is acceptable
or whether every journey must be its own transaction.

### 3.3 Delegated attestations

§11 requires that a host never buys tokens, holds currency, pays gas or owns a wallet, and cites
EAS delegated attestations as the mechanism. Worth being precise: **in Phase 2 the host never
attests at all.** The service attests; the host does nothing. §11's requirement is satisfied by
the host having no role, which is stronger than delegation.

Delegation becomes relevant in Phase 5, where a Host Certificate is a statement *by* the encounter
about the host. The plan notes the mechanism and does not build it.

### 3.4 What must never go on-chain, checked rather than intended

§10's list, and Phase 1 already did the hard part: the lineage names `host_number` (§34) and never
an account, a display name, a credential or anything derived from them
(`server/src/wanderer.js:479-482`). Phase 2 keeps that and adds a test that *scans the published
payload for the private values it should not contain* (line 28) rather than reasoning that it
cannot contain them.

Two subtleties that are not on §10's list and should be:

- **`host_number` is not identifying, but a complete public list of them is a movement trace** of
  the object — every journey, its start, its end, in order, forever. §34 wants the number to be part
  of the story and §32 wants the Passport to show journey counts, so this is very likely fine. It is
  a product call about the public record, not an engineering one (§11.3).
- **Today's `state_hash` must not be published, and the reason is the next section.**

---

## 4. State fingerprints — the one place the current design is wrong for a public chain

### 4.1 What `state_hash` is now

```js
const stateHash = hash(this.stateLines(id))              // wanderer.js:185
stateLines(id) { return ...SELECT seq, epoch, host_number, line, at FROM state... }
```

`line` is the host's own text — Phase 0's entire interaction is a host writing a line to the
Wanderer. So `state_hash` is **a cryptographic commitment over host-authored content.**

Inside our own database that is fine and useful. On an immutable public ledger it is a privacy
defect, and a well-known one: a hash is not a redaction when the input is guessable. Anyone
holding a candidate line can recompute the hash and learn whether they guessed right — a confirmation
oracle for private content, published permanently, on a record designed so that nobody can ever take
it back. §41 says so almost in these words, requiring the manifest to *"exclude host-identifying/
private material"*, and cites the EDPB's July 2026 blockchain guidance on pseudonymous identifiers
that remain linkable.

### 4.2 What replaces it

§41's own example is a manifest of **version counters, not content**:

```text
STATE MANIFEST 9215
wanderer: W-001   epoch: 9215   protocol: 1
visual-state-version: 847   behavior-state-version: 302
memory-manifest-version: 18491   previous-state: ...
```

So Phase 2 introduces two distinct values that today's single `state_hash` is confusing:

```text
manifest_hash    sha256 of the §41 manifest: identifiers and monotonic counters, no content,
                 no host text, no host_number.  →  PUBLISHED. This is §9's stateManifestHash.

content_hash     sha256 over the actual state lines, salted with a per-Wanderer secret that
                 never leaves the service.     →  NEVER PUBLISHED. Kept for §66.10's "stale
                 states are rejected" and for checkpoint integrity, where it already earns
                 its keep.
```

The salt is what makes the private one safe to keep: without it, a stolen database plus a guessed
line is the same oracle, and §39 already asks for encrypted sensitive storage we do not have.

`epochs.state_hash` is a **signed** field, so this is not a rename — it changes what the signature
covers, and an old chain cannot verify against new code. The store is already at
`SCHEMA_VERSION = 2` and refuses a mismatched file by name rather than migrating it
(`store.js:51-68`). Phase 2 is version 3 and takes the same route: refuse, re-mint deliberately.
There is still no real data.

### 4.3 The regression this creates a permanent test for

Line 24 exists to make this unforgettable: **no value published by any Phase 2 surface may be a
hash over host-authored text.** It is the kind of mistake that gets reintroduced by a helpful
refactor eighteen months from now, and by then the ledger is immutable.

---

## 5. The components

### 5.1 Genesis registry — `server/src/genesis-registry.js` (new)

The root of trust, and the answer to §1.1.

A Genesis attestation is made on-chain by the Genesis authority and binds, permanently:

```text
wandererID          W-001
publicKey           the Ed25519 public key the lineage must verify against  ← the missing piece
genesisAt           timestamp
initialManifestHash the epoch-0 manifest
protocolVersion     1
issuer              the Genesis authority address
```

Verification order inverts. Today: *read our key, check the chain.* After Phase 2:

```text
1. wandererID  →  Genesis attestation issued by the KNOWN authority address
2. attestation →  publicKey
3. publicKey   →  walk the local epoch chain
4. epoch chain →  agrees with the attestation chain in both directions (§2.4)
```

Step 1 is what the clone cannot do. It can mint a Wanderer, sign a perfect chain and serve a
confident verifier — and it cannot make the Genesis authority say its key is W-001's, because it
does not hold that authority's key.

**Scarcity (§6.1)** falls out of the same structure: the registry *is* the set of Genesis
attestations from the authority address. "Are there exactly 100?" is a question about that set,
answerable by anyone, and W-101 requires the authority to attest it. Line 6 tests that W-101 cannot
be conjured. Note §5.2's launch quantity remains undecided and Phase 2 decides nothing about it —
the architecture supports Model A and Model B identically, because the registry does not care how
many rows it has.

**§38's key handling, stated bluntly.** The Genesis signing key must never be in browser
JavaScript, shipped, committed, or kept as an ordinary plaintext server secret. Phase 2 runs on a
**testnet key**, held in a file outside the repository, loaded from the environment, and reported as
what it is: not production, and not a thing to reuse. §38.1's HSM, restricted signing service,
2-of-3 approval and offline recovery are documented as the production path and **not built** — they
are operational architecture for a service that has somewhere to deploy, and 2-of-3 is a policy
decision about who the authorities are (§11.4).

### 5.2 Attestation chain — `server/src/attest.js` (new)

Builds the §9 record for an epoch, places it, records it in `attestations`, and re-reads it back
to confirm it landed. That last step is not ceremony: Directive 009 Part A exists because a report
that was committed and never pushed reported success, and an attestation that was signed and never
placed would do the same thing.

Two operations: `attestEpoch(wanderer, epoch)` — off-chain, immediate — and `anchor(wanderer)` —
on-chain, batched, committing to every unanchored off-chain attestation. Neither is in the custody
path (§3.2); both are idempotent, because a retry that double-attests an epoch has invented a fork.

### 5.3 The chain adapter — `server/src/ledger.js` (new), and the deviation it represents

This needs flagging clearly, because it is a compromise and Phase 1 made the same one.

An acceptance suite that talks to a live testnet is a suite that fails when an RPC endpoint is slow,
a faucet is dry or a chain reorganises. The suite runs unattended in the relay watcher every time a
directive lands. Making it depend on public infrastructure makes fifty currently-deterministic lines
flaky, and a flaky acceptance suite stops being read.

So `ledger.js` is one interface with two backends:

```text
LocalLedger    an in-process, EAS-shaped ledger. Deterministic. Real Ed25519/secp256k1
               signatures, real UIDs, real refUID semantics, real rejection of malformed
               input. No network. THE SUITE RUNS ON THIS.

EASLedger      @ethereum-attestation-service/eas-sdk against Base Sepolia. Real
               transactions, real gas, real block timestamps. A SEPARATE, MANUALLY RUN
               INTEGRATION SCRIPT RUNS ON THIS, and its output goes in the report.
```

This is exactly the shape of Phase 1's test authenticator (Directive 008 §4): the server half is
real, the counterparty is a stand-in, the suite stays headless, and the real ceremony has a named
later home. The precedent is deliberate and so is the risk — **a local ledger cannot prove we got
the SDK right**, only that our logic is right. That is why the integration run is a deliverable
(line 31) and not an optional extra, and why §11.1 asks whether this split is acceptable.

### 5.4 Public verification — `server/src/verify-public.js` (new)

The independent verifier, and the acceptance suite's centre of gravity. Its defining property is
what it is **not allowed to touch**: not `wanderers.public_key`, not `wanderers.private_key`, not
any private table. Its only inputs are a Wanderer ID, the ledger, and the public epoch list.

Enforced structurally, not by discipline — it takes a ledger handle and an HTTP endpoint, and has
no database handle to misuse. Line 29 runs it against a service whose store it cannot open.

`verifyLineage` stays where it is and gains the registry check, so the service's own answer and an
outsider's answer are computed by different code and must agree. Two implementations that agree are
worth more here than one shared helper, because the thing being tested is whether the answer
depends on who is asking.

### 5.5 Living Mark — `server/src/living-mark.js` (new)

§33, and it is PROPOSED rather than DECIDED, so this builds the mechanism and none of the art.

The one design rule that matters: **the mark is a rendering of the proof, never a carrier of it.**
A mark that contains its own claim of authenticity is a sticker, and stickers are forged. So the
mark is a pure deterministic function of facts a verifier independently holds:

```text
mark = f(wandererID, genesisAttestationUID, currentEpoch, manifestHash, protocolVersion)
```

A verifier recomputes it and compares. Tampering with the mark (§66.16's explicit threat) is
detected because the recomputation disagrees; a clone's mark differs because its Genesis UID and
head differ (lines 25-27).

Phase 2 renders it as deterministic text and glyph indices, tested by equality. §66.14 forbids Wisp
integration before the infrastructure works, so the animated, evolving mark belongs in Phase 4 with
the body. Which of the ~400 existing glyphs it draws on, and whether it visually evolves, are design
decisions (§11.5).

### 5.6 Lineage viewer — `server/src/index.js` routes + `client/src` (extended)

Read-only, public, authenticated by nothing, naming nobody. It shows the Genesis record, the epoch
list with events and journey numbers and timestamps and attestation UIDs, the verification verdict
with its reasons, and the Living Mark.

**It is not the Passport.** §31's Passport — age, journey count, countries, "Learned a melody" — is
Phase 5, and it needs Phase 3's memory and consent model to exist before it can show anything
truthful. The Phase 2 viewer is the technical lineage view underneath it: the thing a sceptic reads,
not the thing a host shows a friend. Conflating them would put unproven claims on a public page.

It renders **whatever the verifier says, including refusal.** A viewer that can only draw
`AUTHENTIC ✓` has not been tested; the clone must produce a page that says so, with the reason
(§11.8 asks how a counterfeit should be *described*).

---

## 6. The acceptance tests

Thirty-one lines, in the established style: every line either passes or is DENIED. Phase 0's 16 and
Phase 1's 34 stay green alongside, for a suite of 81.

Lines 7-10 are the success condition. Line 7 is the line that **fails today** — §1.1 shows it
failing.

```text
GENESIS REGISTRY AND SCARCITY (§5.1, §6.1, §66.12)
  1  W-001 has a Genesis attestation issued by the Genesis authority
  2  the attested public key is the key the lineage verifies against
  3  minting W-002 requires Genesis authority -- a host session is DENIED
  4  a Wanderer minted with no registry entry is reported NOT authentic
  5  a Genesis attestation signed by a non-authority key is DENIED
  6  the registry enumerates exactly the attested set -- W-101 is absent and cannot be conjured

THE CLONE -- THE SUCCESS CONDITION (§13, §66.12)
  7  a clone service's own W-001 is reported NOT authentic: key is not the registered key
  8  ...and the clone's own chain is internally valid, so line 7 proves the registry did the work
  9  a clone presenting the REAL Genesis attestation UID with its own key is DENIED
 10  a clone holding a verbatim copy of epochs 0..N is distinguished by the HEAD it cannot extend
     (stated as the honest limit of §1.2, not as a stronger claim)

THE ATTESTATION CHAIN (§8, §2.4, §66.12)
 11  every epoch has exactly one attestation
 12  attestation N's refUID is attestation N-1's UID, from Genesis forward
 13  the attestation commits to the epoch's local chain link, sha256(signature)
 14  altering a local epoch is detected by disagreement with the attestation chain
 15  reordering local epochs is detected
 16  an epoch present locally but never attested is reported as a gap
 17  an attestation with no corresponding local epoch is reported as the reverse gap
 18  a malformed attestation is DENIED, not ignored (§66.16)
 19  an attestation naming a different wandererID is DENIED

STATE FINGERPRINTS (§41, §5 above)
 20  the published manifest contains no host text, no account, no credential, no host_number
 21  the manifest hash changes when state advances
 22  two Wanderers at identical versions do not collide
 23  a forged state manifest is detected (§66.16)
 24  NO published value is a hash over host-authored text -- the §4.3 privacy regression line

THE LIVING MARK (§33, §66.16)
 25  the mark is recomputed from public facts alone: same facts, same mark
 26  a tampered mark does not verify -- the verifier recomputes and disagrees
 27  the clone's mark differs from the real W-001's

PUBLIC VERIFICATION AND THE VIEWER (§66.12, §32, §37)
 28  the viewer payload is scanned for every private value and contains none of them
 29  the independent verifier, with NO database access, reaches the service's verdict
 30  verification works while held, and while held by nobody

OPERATIONS
 31  measured Base Sepolia gas for a Genesis attestation and for one anchor, recorded in the report
```

Line 31 is deliberately a measurement rather than a threshold. This plan does not quote gas prices
it cannot verify — there is no network access in the session writing it — and inventing a figure to
look thorough is worse than measuring one later.

---

## 7. §66.17 change documentation

Three changes qualify. Blockchain and signing are both on §66.17's list.

### 7.1 The root of trust moves outside the service

```text
ORIGINAL STATE
  verifyLineage re-walks the chain and re-checks signatures against wanderers.public_key,
  read from the service's own database. Two independent services each minting W-001 both
  report authentic=true (§1.1, demonstrated).

PROPOSED CHANGE
  W-001's public key is established by an on-chain Genesis attestation from the Genesis
  authority. Verification resolves the key from the registry FIRST, then walks the chain.
  An independent verifier with no database access performs the same check.

REASON
  §67 Phase 2's success condition requires independent distinction from a clone. §66.12
  requires that a copied Wanderer cannot produce valid current provenance. Neither is
  satisfiable while the key and the chain come from the same place.

FILES AFFECTED
  server/src/genesis-registry.js (new)   server/src/verify-public.js (new)
  server/src/attest.js (new)             server/src/ledger.js (new)
  server/src/living-mark.js (new)        server/src/wanderer.js (genesis, verifyLineage)
  server/src/store.js (attestations table + triggers, SCHEMA_VERSION 3)
  server/src/index.js (public routes)    client/src (viewer)
  server/src/acceptance-phase2.js (new)  package.json (scripts, deps)

SECURITY / PRIVACY EFFECT
  + a counterfeit service can no longer produce a passing verdict
  + scarcity becomes checkable by a third party (§6.1)
  + tampering with the local lineage becomes externally detectable
  - a NEW high-value key exists: the Genesis authority's chain key. §38 governs it; Phase 2
    uses a testnet key held outside the repo and claims nothing more (§5.1)
  - a permanent public record is created. Mistakes are not erasable. This is why §4 changes
    what gets fingerprinted BEFORE anything is published, not after
  - a new external dependency (an RPC endpoint) enters verification, though NOT custody (§3.2)

TESTS REQUIRED
  lines 1-19, 28-30. Line 7 is the condition; line 8 proves line 7 for the right reason.

RESULT
  to be recorded in REPORTS.md after implementation.
```

### 7.2 What a state fingerprint commits to

```text
ORIGINAL STATE
  epochs.state_hash = sha256 over state rows including `line`, the host's own text. It is a
  signed field of the lineage.

PROPOSED CHANGE
  Split into manifest_hash (§41 identifiers and monotonic counters; PUBLISHED and signed) and
  a salted content_hash (kept private, never published). SCHEMA_VERSION → 3; an older store is
  REFUSED by name, as version 2 already refuses version 1, rather than migrated (§66.6).

REASON
  §41 requires the manifest to exclude host-identifying and private material and cites the
  EDPB's July 2026 blockchain guidance. A hash over guessable host text, published on an
  immutable ledger, is a permanent confirmation oracle for private content (§4.1). §10 forbids
  conversation content on-chain, and a commitment to content is a form of the content.

FILES AFFECTED
  server/src/wanderer.js (genesis, openEpoch, checkpoint, verifyLineage)
  server/src/store.js (SCHEMA_VERSION, wanderers salt column)
  server/src/acceptance.js and acceptance-phase1.js (state_hash assertions)

SECURITY / PRIVACY EFFECT
  + host-authored content is no longer committed to any published value
  + the private commitment is salted, so a stolen database is not an oracle either
  - what the epoch signature covers changes; Phase 0 and Phase 1 chains cannot verify against
    Phase 2 code. Acceptable: there is no real data and every store is a test artifact
  - two hashes where there was one is a chance to publish the wrong one. Line 24 exists solely
    to catch that, permanently

TESTS REQUIRED
  lines 20-24, plus all 50 existing lines staying green.

RESULT
  to be recorded in REPORTS.md after implementation.
```

### 7.3 Attestation records, and NOT loosening the lineage triggers

```text
ORIGINAL STATE
  epochs is append-only by trigger; closed_at is the only field a written row may gain
  (store.js:191-214).

PROPOSED CHANGE
  Attestation UIDs go in a new `attestations` table permitting no updates and no deletes.
  epochs and its triggers are UNCHANGED.

REASON
  An attestation UID arrives after the epoch row is written, so the obvious epochs.attestation_uid
  column would require permitting a second mutable field on the one table whose immutability is
  the argument of the whole lineage. The convenience is not worth the guard (§2.6).

FILES AFFECTED
  server/src/store.js   server/src/attest.js   server/src/verify-public.js

SECURITY / PRIVACY EFFECT
  + the lineage's append-only guarantee is untouched, so nothing in Phase 1's proof weakens
  + attestation records are themselves append-only; a wrong one is corrected by another row
  - a Wanderer may briefly have epochs that are real and not yet attested. That is the
    asynchronous design of §3.2, not a defect, and lines 16-17 test that the gap is REPORTED
    rather than hidden

TESTS REQUIRED
  lines 11-17.

RESULT
  to be recorded in REPORTS.md after implementation.
```

---

## 8. Built with

```text
@ethereum-attestation-service/eas-sdk    §8's named candidate. Integration script only.
ethers                                    required by the SDK.
node:crypto                               everything the LocalLedger and the marks need.
better-sqlite3, express                   already here.
```

Base Sepolia for the testnet. Faucet ETH, so the measured cost in real money is zero, and line 31
records the gas anyway because the mainnet decision needs a number.

Two dependencies enter the project for a signing path, which is the category §66.15 flags. Both are
confined to `EASLedger`; nothing in the suite's path imports them. Version pinning and a look at
what they pull in belongs in the implementation, not in a plan that guesses.

---

## 9. Migration

`SCHEMA_VERSION 3`. An older store is refused by name and version with an explicit message, as
version 2 already refuses version 1 (`store.js:61-68`). No automatic migration, no automatic drop —
§66.6 forbids destructive migration without review, and re-minting is a decision for whoever owns
the file.

Every existing store under `data/` is a test artifact re-created by the suite. Nothing is lost. If
that ever stops being true, this paragraph stops being adequate and the answer becomes a real
migration with a real review.

---

## 10. Deliberately out of scope for Phase 2

- **The Passport** (§31, §32) — Phase 5. The Phase 2 viewer is the technical lineage view (§5.6).
- **Host Certificates and Verifiable Credentials** (§35, §36) — Phase 5. They describe the host,
  and Phase 2 publishes nothing about hosts.
- **Mainnet.** Which chain, and when, is a product and cost decision (§11.1).
- **The animated, evolving Living Mark** — Phase 4, with the body (§66.14).
- **The number of Genesis Wanderers** (§5.2) — §66.18 forbids deciding it. The registry supports
  Model A and Model B without changing a line.
- **§38.1's 2-of-3 approval, HSM and offline recovery** — documented as the production path,
  not built. Operational architecture for a service with somewhere to deploy (§11.4).
- **`STATE_COMMIT` and `RETIREMENT` events** (§9) — nothing asks for them yet (§11.6).
- **Independent security review** (§66.15) — not out of scope, *outstanding*, and Phase 2 makes it
  more necessary rather than less: it adds a signing key, a public ledger and permanent records.
  Nothing in this plan substitutes for it.

---

## 11. Open questions for Lonnie

Eight, and none of them should be answered by me. Numbers 1, 2 and 3 change what gets built; the
rest change details.

**11.1 The chain, and the local-ledger split.** Phase 2 proposes Base Sepolia for the real
integration and a deterministic in-process ledger for the acceptance suite (§5.3). This mirrors
Phase 1's test authenticator, and it means the suite proves our logic while a separate manual run
proves the SDK. Acceptable? And is Base Sepolia the right testnet — the mainnet choice can wait for
Phase 7, but the testnet should ideally be the same family.

**11.2 Is every journey its own transaction?** §8's diagram reads as one attestation per journey.
§3.2 proposes off-chain per epoch plus periodic on-chain anchors, because a transaction in the
custody path breaks when an RPC does, and per-epoch cost grows forever. The trade-off is a window
in which epoch ordering rests on our signature rather than on a block. Is that window acceptable,
and if so, how long? "Every journey is on the blockchain" may be worth real money as a story.

**11.3 Does the public lineage list journey numbers?** §34 wants the number to be part of the
story. A complete public list of every journey number with start and end times is also a permanent
movement trace of the object, and any host who mentions their own number links themselves to a slice
of it — their choice (§37), but a choice we would be creating. Show all, show counts only, or show a
window?

**11.4 The Genesis authority key, for now.** Phase 2 proposes a testnet key in a file outside the
repository, loaded from the environment, reported as not-production. Confirm — and note that this
is the same open question Phase 1 ended on, arriving with an actual key attached: the custody
authority is currently "anyone with code in this process", and Phase 2 would make the Genesis
authority "anyone with that file."

**11.5 The Living Mark's vocabulary.** Which of the ~400 existing glyphs, and does the mark
visually evolve as the Wanderer does? §33 says it *can*; degree of evolution is §66.18's list.

**11.6 `STATE_COMMIT` and `RETIREMENT`.** §9 proposes both. The model has no state-commit event
(state accumulates within an epoch and is fingerprinted at the boundary), and nothing in §67 asks
for retirement. Leave both unimplemented?

**11.7 `memory-manifest-version` in the manifest.** §41's example includes it; memory is Phase 3.
Proposal: reserve the field, set it to 0, so the manifest's shape does not change when Phase 3
lands — a manifest whose shape changes changes every hash after it.

**11.8 How should a counterfeit be described?** §13.2 says the hacked instance "has become a fork
— it is not the Wanderer." Should the verifier and viewer say that, in those words — naming a fork
of W-001 at epoch N — or refuse flatly with no story? It reads as a technical question and it is
a narrative one, and whatever it says will be quoted.

---

## 12. What Phase 2 will have proved, if it works

Phase 0: a copy of the client is not the Wanderer.
Phase 1: no host, however hostile or disconnected, can keep, retake or destroy it.
Phase 2: **and a stranger who trusts none of us can tell which one is real.**

The line that carries it is line 7, and today it fails.

---

## Deviations and things to know

**1. Part A's diagnosis differs from the directive's.** The directive says the push failed
silently; the transcript says no push was attempted. I built what was asked — verify, retry once,
log loudly — because it is the right fix for both, and the verification half is what actually
catches this. Recorded because the difference is the interesting part: the failure was an omitted
step reported as success, which is the same shape as the failure Phase 2 exists to prevent on a
ledger nobody can edit.

**2. The uncommitted-changes ALARM does not fail the cycle.** It shouts and marks the cycle WITH
WARNINGS, but returns success if the committed work landed. Failing on any stray file would produce
a failed unit every minute for a scratch file, and an alarm that cries wolf is worse than none.

**3. `watch.sh` now exits non-zero on an incomplete cycle**, so the systemd unit will show as
failed. That is intended — a silent failure was the whole complaint — but it does mean
`systemctl --user status` will look alarming after a genuinely failed delivery. The timer is
unaffected and keeps firing.

**4. A self-editing script is a hazard I only half-avoided.** See Part A's closing note. The watcher
amends itself while running, my edits shifted ninety lines above its resume point, and it survived
because the editing tool renames rather than writes in place. The rule is now written at the top of
`watch.sh`, and the running instance was verified to still hold the original bytes. Worth stating
plainly: had I written that file with a shell redirect, this cycle would probably have ended by
executing a fragment of its own new source.

**5. No gas figures are quoted.** Part B needed testnet costs, and this session had no network
access to verify current EAS deployments or fees. Rather than write plausible numbers into a plan,
the plan makes measuring them acceptance line 31. An invented figure would have looked more
thorough and been worth less.

**6. The plan proposes a deterministic in-process ledger for the suite**, with the real EAS SDK
exercised by a separate manual integration run — the same compromise Phase 1 made with the test
authenticator, and it has the same weakness: it cannot prove the SDK is used correctly. Flagged in
the plan (§5.3) and open question 11.1 asks whether the split is acceptable.

**7. Attestation is asynchronous to custody in the proposal.** A lease that cannot be issued until
a transaction confirms is a lease that fails when an RPC endpoint does, which gives away Phase 1's
central argument. It costs a window in which epoch ordering rests on our signature rather than on a
block. Open question 11.2, and it may be a product decision rather than an engineering one — "every
journey is on the blockchain" is worth something as a story.

**8. `SCHEMA_VERSION` will go to 3 and old stores will be refused, not migrated** — the same route
version 2 took. Every store under `data/` is a test artifact. If that stops being true, that
paragraph of the plan stops being adequate.

## Still not done, and named so it is not assumed done

- **Independent security review (§66.15).** Outstanding, and Phase 2 makes it *more* necessary: it
  adds a signing key, a public ledger, and records that cannot be withdrawn.
- **Directive 003's own report** — still absent from this file, as noted in the Directive 004b
  cycle. Say the word and I will backfill it.
- The Phase 1 open question stands and Phase 2 sharpens it: the custody authority is "anyone with
  code in this process", and a Genesis chain key would make the Genesis authority "anyone with that
  file." Open question 11.4.

## Status

**Part A: done, tested, live.** **Part B: proposal only. Nothing built. STOP for approval**, per
Directive 009 step 3 and §66.17.

Suite unchanged and green at 50 (16 + 34), 0 failed, exit 0. `a975413` in CC-Wanderer.

Eight questions in §11 of the plan are waiting on you. Three of them decide what gets built.

---

# Directive 010 + amendment — real WebAuthn, then Phase 2 on a real chain

**Status: both parts built, all three suites green. Phase 0 16 + Phase 1 39 + Phase 2 32 = 87
passed, 0 failed, exit 0.** `c6c0251` in CC-Wanderer (Part A `6f323ee`, Part B `c6c0251`).

Two deviations, both recorded in the suite output itself rather than only here. Neither is a
failure; one of them is a loss of coverage and is written up as such.

---

## 0. The two verifications the directive demanded FIRST

Both amendments say the same thing: prove the tool works on this machine before building on it,
and if it does not, report rather than work around. So nothing below was written until these
passed.

**Real WebAuthn in a real browser — PASSED.** Google Chrome 151.0.7922.71 is on this machine.
Driven through `puppeteer-core` and the CDP `WebAuthn.*` virtual authenticator, it produced a real
credential (194-byte attestation object) and a real assertion (71-byte signature) over
`http://localhost`, with `clientDataJSON` written by the browser. A probe then established exactly
which of the suite's attacks the real stack can perform:

```
counter on 3 successive assertions: 2 3 4
getCredentials exposes: credentialId, isResidentCredential, rpId, privateKey, userHandle,
                        signCount, backupEligibility, backupState, userName, userDisplayName
after rewinding signCount to 0, next assertion signs: 1
localhost page claiming rp.id=evil.example -> DOMException
page on http://evil.example making a credential for rp.id=evil.example -> q2avNh8u7Xr6iDnCoq...
```

**Anvil + real EAS contracts — PASSED.** Foundry was not installed; it is now (anvil/forge/cast
1.7.1). The real `SchemaRegistry.sol` and `EAS.sol` deploy to Anvil from the published
`@ethereum-attestation-service/eas-contracts` artifacts, and the real SDK drives them:

```
chain id: 31337
SchemaRegistry deployed: 0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512
EAS deployed:            0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0
EAS version reported by the contract: 1.4.0
schema registered, attestation placed, read back, decoded, attester matches
VERIFY-EAS: OK
```

Two obstacles were hit during verification and are worth recording because both are the kind of
thing that silently produces a wrong answer rather than an error:

1. **The EAS SDK's ESM bundle does not load at all** under Node's ESM resolver — it has a broken
   named import of `lodash`. Its CommonJS build is the same published SDK and is used instead.
   This is a packaging detail, not a substitute for the SDK.
2. **`ethers` caches `eth_getTransactionCount` for ~250ms, and Anvil mines faster than that**, so
   two sends in a row both claimed nonce 0 and the second was rejected with "nonce has already been
   used". `NonceManager` fixes it. The same cache later made a naive gas measurement silently
   return zero — see §4.

---

## 1. Part A — the test authenticator is deleted

`server/src/test-authenticator.js` is **gone**, not deprecated. It held an exportable private key,
which is the one property a real authenticator exists to make impossible, and the file said so
itself. Both suites now drive Chrome's own WebAuthn implementation (`server/src/browser.js`).

**The login surface arrived early, from Phase 4**, because the amendment asked for it and because a
ceremony no person can perform is a ceremony nobody has tested. `server/src/login.js` serves a
minimal page that runs the real ceremony in the browser; `client/src/login.js` is the CLI that
waits for it. The handoff uses two values that travel differently:

- the **ticket** goes in the URL, where it is visible in shell history and window titles;
- the **claim** never leaves the CLI and is the only thing that redeems the session.

Whoever reads the ticket can at most deliver a session *into* the handoff. Only the process holding
the claim can take one out, once, before it expires. Line 15 of the Phase 1 suite drives the whole
thing end to end — real page, real ceremony, real platform passkey, session arriving in the
terminal — and lines 16 and 17 attack the handoff.

### What changed about what is proved, and it is not all in our favour

Three attacks moved, and two of them got **stronger**:

| attack | before | now |
|---|---|---|
| phishing origin | software authenticator wrote a false origin | a page **really served** at `http://evil.example` runs the whole ceremony with our live challenge; Chrome writes that origin itself; the **server** refuses it |
| assert a localhost passkey from that origin | expressible | the **browser refuses** — a credential is bound to its RP ID. The attack cannot be mounted at all |
| foreign RP ID from a localhost page | expressible | the **browser refuses** with `SecurityError` |
| rewound sign counter | a fake number in a fake message | the **authenticator's own counter**, wound back through CDP, signed by Chrome |

The server's refusal on the first one reads well, and it is the real error text:

```
5  Real ceremony performed at a phishing origin, relayed to us  DENIED
   (registration was not verified: Unexpected registration response origin
    "http://evil.example:35191", expected "http://localhost:35191")

9  Sign counter goes backwards (cloned authenticator)  DENIED
   (assertion was not verified: Response counter value 1 was lower than expected 2)
```

### DEVIATION 1 — a rule that is now untested

**Chrome's virtual authenticator always advances its sign counter.** The commonest passkey in the
world — a synced one — reports 0 for ever, and `webauthn.js` deliberately accepts that, because
refusing it would lock out most passkeys that exist. The real stack **cannot produce that case**:
the floor is 1, and rewinding to 0 before every assertion makes it sign a constant 1, which the
counter rule correctly treats as a clone signal.

So that line is gone. The server rule is **unchanged and now untested**. The old software
authenticator could fake it; per Directive 010 this suite does not. It is printed as a DEVIATION in
the suite output every run, so it cannot quietly become "covered". If you want it back, the honest
options are a different browser engine, or a test that calls the counter policy directly and is
labelled as not being a browser ceremony — say which and I will build it.

Three smaller things the real browser forced, each a genuine property rather than a workaround:

- **An origin has a port in it.** Each service now tells its relying party the origin it is actually
  listening on, and the restart test comes back up on the *same port* — otherwise every credential
  in the browser would belong to a different relying party than the service that issued it.
- **WebAuthn refuses to run in an unfocused tab**, and each authenticator has its own tab (one tab
  per authenticator: Chrome allows only one `internal` virtual authenticator per browser, and a tab
  holding several makes credential selection ambiguous). The suite brings the right tab forward.
- **Stopping a service dead now means dropping the browser's keep-alive sockets**, which is what the
  restart test is testing. The test client reconnects once on a dead pooled socket, as any real
  client does; a *refusal* is never retried.

---

## 2. Part B — Phase 2, and no test doubles for the ledger either

`PHASE2_PLAN.md` §5.3 proposed a deterministic in-process ledger for the suite with the real SDK
exercised only by a separate manual run. **Directive 010 rejected that, and it is not built.** There
is one backend: EAS's own contracts, driven by EAS's own SDK. The acceptance suite starts a real
Anvil node, deploys the real contracts to it, and runs against them. Base Sepolia differs by the
RPC URL and by nothing else — there is no branch anywhere in the attestation path.

Built: `ledger.js`, `genesis-registry.js`, `attest.js`, `manifest.js`, `living-mark.js`,
`verify-public.js`, `viewer.js`, schema version 3, and `acceptance-phase2.js`.

### The success condition

> W-001 must be independently distinguishable from a clone.

The clone in the suite is not a broken copy. It is a complete, honest counterfeit: its own service,
its own database, its own W-001, its own key, its own perfectly signed chain — everything a
determined counterfeiter holding our source code would have. What it does not have is the Genesis
authority's key.

```
 5  The clone's own chain is internally valid — so the next line is the registry's work ✓
    the clone's verifier says AUTHENTIC about the clone, over 2 links
 6  The clone, judged by the independent verifier   DENIED  (NOT THE WANDERER)
    it says: This is not W-001. It is not registered by the Genesis authority,
             so it was never W-001 to diverge from.
 7  A clone presenting the REAL Genesis UID with its own key  DENIED
    (epoch 0 is not signed by the key the Genesis authority attested)
 8  And the real W-001, judged by the same verifier with no database ✓  3 links, 3 attestations
```

Line 5 is what makes line 6 mean anything: the clone is internally flawless, so line 6 is the
registry doing the work and not a bug in the clone. Line 7 is the cleverer clone that stops claiming
its own Genesis and presents the *real* UID — it gets past the authority check and dies one step
later, on the key it cannot have.

The verifier (`verify-public.js`) **has no database handle at all**. Not the private key, not the
public key, not the salt — it cannot reach them structurally, rather than by being careful. Its
inputs are a Wanderer ID, a chain, an authority address and a public HTTP endpoint. Line 27 asserts
that property against the source text so a helpful refactor cannot quietly hand it one.

### §11.8 — how a counterfeit is described

In the spec's own words, and the wording distinguishes two different things, because it will be
quoted:

- something that **never was** W-001: *"This is not W-001. It is not registered by the Genesis
  authority, so it was never W-001 to diverge from."*
- something that **was and diverged**: *"This has become a fork of W-001 at epoch N — it is not the
  Wanderer."*

Calling the first one "a fork at epoch 0" would describe a relationship it does not have.

---

## 3. The state-fingerprint change, which was a real defect

`epochs.state_hash` was `sha256(the host's own written lines)` and it was about to be published.
**A hash is not a redaction when the input is guessable**: anyone holding a candidate line could
recompute it and learn whether they guessed right — a confirmation oracle for private content,
published permanently, on a record designed so nobody can withdraw it.

Two values now, going to two places:

- **`manifest_hash`** — over §41's manifest of identifiers and monotonic counters. No host text, no
  account, no credential, no host number. **Published.** This is what an epoch now signs.
- **content hash** — over the actual lines, **salted** with a per-Wanderer secret. **Never
  published.** Keeps earning its keep in checkpoint integrity and §40 recovery. The salt is what
  makes it safe to keep at all: without it, a stolen database plus a guessed line is the same
  oracle, aimed at whoever steals the file instead of whoever reads the chain.

`memory-manifest-version` is reserved at 0 as directed (§11.7), along with the visual and behaviour
counters — a manifest whose *shape* changes when Phase 3 or Phase 4 lands would change every hash
after it, and those hashes are on a ledger nobody can edit.

Line 23 is the permanent regression guard §4.3 asked for: it recomputes every content hash, salted
and unsalted, over every prefix of the state and every individual line, and scans the entire
published record **and the rendered viewer HTML** for any of them. Zero found. This is the mistake a
well-meaning refactor reintroduces in eighteen months.

Schema is version 3. Old stores are **refused by name and version, not migrated** — the same route
version 2 took, and §66.6 forbids destructive migration without review. Every store under `data/` is
a test artifact.

---

## 4. Operations, and the gas figure that is not the one asked for

### DEVIATION 2 — gas is measured on the local chain, not Base Sepolia

Plan line 31 asked for **measured Base Sepolia gas**. It is not measured, because that needs a
funded testnet key and there is none on this machine. What is measured is gas on the real EAS
contracts on the local chain — the same bytecode, therefore the same gas:

```
32  Gas for a Genesis attestation and for an anchor is measured, not guessed  ✓
    Genesis 482050 gas, anchor 321288 gas — read off the receipts
```

**No currency figure is quoted, because gas *price* was not measured.** Same principle as the
Directive 009 report: an invented number looks more thorough and is worth less. Give me a funded
Base Sepolia key and line 31 becomes the real thing.

The first attempt at this measurement silently returned zero, because it diffed block numbers and
ethers caches `eth_blockNumber` inside the window Anvil mines in. It now reads `gasUsed` off the
receipt. Worth recording as the second time in this directive that a cache produced a plausible
wrong answer rather than an error.

### The key, stated plainly

Testnet key from the environment, outside the repository, as §11.4 directed. On a local dev chain it
falls back to Anvil's first well-known account, which is a *publicly known private key on a
throwaway chain* and is reported as exactly that. §38.1's HSM, restricted signing service, 2-of-3
approval and offline recovery are documented as the production path and are **not built**. Today the
Genesis authority is "whoever holds that file", and that is the honest description.

---

## 5. Things to know, and one open tension

**1. Journey numbers, §11.3, and a conflict the directive could not have foreseen.** You said counts
only. Implemented: nothing on the ledger carries a journey number (see the epoch schema in
`ledger.js`), and the viewer shows `"N in total"` and never which ones (line 29).

But the **verification feed still carries them**, and it has to: `host_number` is inside the Ed25519
signature over each epoch, so a lineage served without it cannot be verified by anybody, which would
defeat the whole phase. So "counts only" holds for the permanent record and for the human-facing
page, and does not hold for the machine-readable feed a verifier reads. If you want it to hold
there too, the fix is to take `host_number` out of what is signed — a change to the lineage format
and therefore a re-mint, and I have not made it on my own authority.

**2. Attestation is asynchronous to custody, as approved (§11.2).** Custody never waits for an RPC.
The cost is a window in which epoch ordering rests on our signature rather than on a block; the
anchor closes it. `anchor()` is callable every epoch, so the per-journey-on-chain story remains
available as a launch decision.

**3. `STATE_COMMIT` and `RETIREMENT` are not emitted**, per §11.6. The event mapping is explicit in
`attest.js` rather than the model being bent to fit §9.

**4. The Living Mark is a placeholder and says so in its own output.** Mechanism only: it is a pure
function of public facts, so a verifier recomputes and compares. The glyph vocabulary and whether it
evolves are yours (§11.5, §66.18). A mark that carried its own claim of authenticity would be a
sticker, and stickers are forged.

**5. The viewer is not the Passport.** §31's Passport is Phase 5 and needs Phase 3's memory and
consent model before it can show anything truthful.

**6. A local chain is not a public one.** No reorgs, no gas market, no congestion, no adversarial
validators. What the suite proves is the contracts, the SDK, and our use of both.

## Still not done, and named so it is not assumed done

- **Independent security review (§66.15).** Still outstanding, and Phase 2 makes it *more*
  necessary: it adds a signing key, a public ledger, and records that cannot be withdrawn.
- **The synced-passkey counter rule** — untested, per Deviation 1.
- **Base Sepolia gas** — unmeasured, per Deviation 2.
- **Directive 003's own report** — still absent from this file, as noted in the 004b and 009 cycles.
  Say the word and I will backfill it.
