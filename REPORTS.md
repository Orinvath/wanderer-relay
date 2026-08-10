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
