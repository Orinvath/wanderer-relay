# DIRECTIVES — read by Claude Code at task start

## Directive 002 — Phase 0, test 13: lease expiry

In /home/nobara-user/CC-Wanderer:

1. Commit current state before changes (failsafe rule: always commit before any change).
2. Add acceptance test 13: lease expiry.
   - Lease length 3 seconds for this test only (do not change the 60s constant globally).
   - Wait out the expiry.
   - Confirm the holding host is DENIED after expiry.
   - Confirm the service can transition custody to a new host without a release from the expired holder.
3. Run the full acceptance suite (all 13).
4. Commit results.
5. Write the full test output and any deviations to REPORTS.md in this relay repo.

Scope ends there. Nothing else.

## Directive 003 — Relay watcher (after 002 completes)

Set up automatic relay polling on this machine:

1. Create /home/nobara-user/relay-watcher/watch.sh:
   - Every 5 minutes, git pull the wanderer-relay repo.
   - If DIRECTIVES.md changed since last execution (track last-seen commit hash in a state file), launch: claude -p "Read DIRECTIVES.md in the wanderer-relay repo and execute the newest directive per the established workflow. Commit results to REPORTS.md."
   - Log each run to /home/nobara-user/relay-watcher/watcher.log
2. Install as a systemd user timer (5 min interval), enabled and started.
3. Verify claude CLI headless mode works on this machine before wiring the timer; if it fails, report the failure instead of building around it.
4. Commit-before-change rule applies to all directives executed this way, always.
5. Report setup result and one full automated cycle test to REPORTS.md.

## Directive 004 — Automated cycle proof

This directive should be picked up by the watcher, not by manual invocation.
Append to REPORTS.md: "Directive 004 executed via watcher" plus the timestamp of execution.
Nothing else.

## Directive 004b — Retrigger

Directive 004 remains unexecuted. Execute it now: append to REPORTS.md "Directive 004 executed via watcher" with execution timestamp. Nothing else.

## Directive 005 — Expiry closes the epoch

Decisions from Lonnie on the two open questions from test 13:

Q1: Lease expiry closes the epoch immediately. The Wanderer must never sit in a stale epoch — an expired lease means the epoch transitions at that moment, holder becomes none, verifier reflects it. Implement this.

Q2: No cooling period, no re-lease restriction. Testing requires unrestricted lease/release/re-lease by the same hosts. Host-selection and hostage-prevention rules are Phase 1 product decisions — build nothing for them now.

Steps:
1. Commit before changes.
2. Implement Q1. Update affected acceptance tests.
3. Run full suite, all tests.
4. Commit, report results to REPORTS.md.

Scope ends there.

## Directive 006 — Release closes the epoch too

Lonnie's decision: the moment a host lets go — release or expiry, any reason — the epoch closes and their name comes off. An open epoch always names the actual current holder.

1. Commit before changes.
2. Implement for voluntary release, matching the expiry behavior.
3. Update affected tests, run full suite.
4. Commit, report to REPORTS.md.

Scope ends there.

## Directive 007 — Phase 1 begins: Custody

Phase 0 is complete. Proceed to Phase 1 per WANDERER_SPEC.md section 67:
accounts, passkeys, custody leasing, server-authoritative state, forced expiration, release, transfer, recovery. The Wanderer must survive hostile or disconnected hosts.

Per spec 66.17, custody and authentication changes require proposal before implementation:

1. Read the spec's Phase 1 requirements and all sections governing accounts, passkeys, custody, and recovery.
2. Write PHASE1_PLAN.md in CC-Wanderer: the components, how passkeys (WebAuthn) integrate with the existing lease/epoch model, what "hostile or disconnected host" scenarios must be survived, and the acceptance tests that will prove each.
3. Commit the plan. Copy its full text into REPORTS.md.
4. STOP. No implementation until the plan is approved.

## Directive 008 — Phase 1 plan approved, answers to all 8 questions

PHASE1_PLAN.md is approved. Answers, all approved by Lonnie:

1. Recovery: multi-passkey enrolment only; test authenticators for now. Real recovery routes are a pre-launch product decision.
2. Anonymity: passkey and nothing else. No email, no name. Identity policy decided later.
3. @simplewebauthn/server: approved.
4. Login surface: defer real browser ceremony to Phase 4. Phase 1 runs on the test authenticator; suite stays headless.
5. RP ID/origin: localhost. Production domain is Phase 7.
6. Operator: service-local administrative call, no remote surface.
7. Host numbers: per-Wanderer, allocated at lease, never reused. A returning account gets a new number. Confirmed.
8. No limit on Wanderers per account during testing. Product rule later.

Implement Phase 1 per the approved plan:
1. Commit before changes.
2. Build. Run the full acceptance suite (Phase 0 tests must stay green alongside the new Phase 1 tests).
3. Commit, report results and any deviations to REPORTS.md.

Scope: the approved plan only.

## Directive 009 — Watcher push fix, then Phase 2 plan

Part A — fix the silent failure:
The Phase 1 report committed locally but the push to the relay failed silently (status 0, nothing landed). Amend watch.sh: after committing to REPORTS.md, verify the push succeeded; on failure, retry once, then log the failure loudly to watcher.log. Keep a dated backup of watch.sh before editing.

Part B — Phase 2 plan (authenticity, per spec section 67):
Genesis registry, public verification, attestation chain, state fingerprints, Living Mark, lineage viewer. Success condition: W-001 independently distinguishable from a clone.

Per spec 66.17, this touches blockchain and signing — plan first:
1. Write PHASE2_PLAN.md: components, how EAS attestations map onto the existing epoch chain, what runs on-chain vs off, costs/testnet strategy, and the acceptance tests.
2. Commit, copy full text to REPORTS.md, verify the push landed.
3. STOP for approval before implementing.

## Directive 010 — Phase 2 approved with amendments

Lonnie's overriding decision: NO test doubles for the ledger. Build it real. The deterministic in-process ledger from plan section 5.3 is rejected. Instead: run a real local Ethereum node (Anvil) with the real EAS contracts deployed to it, and run the acceptance suite against that — the identical production code path used for Base Sepolia, differing only in RPC URL. Verify Anvil + EAS contract deployment actually works on this machine before building on it; if it fails, report, don't work around.

Answers to the 8 questions:
11.1: Real SDK everywhere per the above. Base Sepolia confirmed as the testnet.
11.2: Off-chain per epoch, periodic on-chain anchors. Custody never blocks on an RPC. Per-journey-on-chain as marketed story is a launch decision; design must support both.
11.3: Counts only in the public lineage. A published ledger cannot be unpublished; showing more can be enabled later.
11.4: Confirmed — testnet key outside repo, from environment, reported not-production.
11.5: Placeholder Living Mark. Glyph vocabulary and evolution degree are Lonnie's, later.
11.6: STATE_COMMIT and RETIREMENT: leave unimplemented.
11.7: Reserve memory-manifest-version at 0.
11.8: The verifier names it in the spec's words: a fork of W-001 at epoch N, not the Wanderer.

Then:
1. Commit before changes.
2. Build Phase 2 per the amended plan.
3. Full suite green (Phase 0+1 tests intact plus Phase 2), all against the real local chain.
4. Commit, report, verify the push landed.

## Directive 010 amendment — no test doubles applies to past features too

Lonnie's rule is retroactive. The Phase 1 test authenticator (exportable-key stand-in) is now also rejected.

Build the real login surface now instead of deferring to Phase 4: a minimal local page performs the real WebAuthn ceremony in a real browser and hands the session to the CLI (the plan's option A). The server-side verification already built stays.

For the automated suite, drive the real browser's own WebAuthn implementation via its automation interface (Chromium DevTools virtual authenticator) — real browser, real WebAuthn stack, no hand-rolled substitute. Verify this works on this machine before building on it; report if it doesn't.

Priority order: real-auth replacement first, then Phase 2 build.

## Directive 011 — Journey numbers resolved by spec; close loose ends

Resolution: Spec section 34 rules. Journey numbers are public story elements ("Host 1,847 of W-001") — only host identity stays private, and identity is never on the ledger. The verification feed carrying numbers is correct. The public viewer may also show journey numbers per spec; update it to do so. No re-mint.

Loose ends:
1. Commit before changes.
2. Update viewer to show journey numbers per spec 34.
3. Backfill the missing Directive 003 report into REPORTS.md.
4. Run full suite, confirm green.
5. Commit, report, verify push landed.

Outstanding items staying open by decision: Base Sepolia gas (needs funded key), synced-passkey counter test, independent security review — all pre-launch, none now.

## Directive 012 — Phase 3 plan: Memory

Phases 0-2 complete. Proceed to Phase 3 per WANDERER_SPEC.md section 67: memory.

Per spec 66.17, memory and consent boundaries require proposal before implementation:

1. Read all spec sections governing memory, retention, forgetting, consent, and what carries between hosts (sections 26 and related).
2. Write PHASE3_PLAN.md: components, how memory attaches to the existing epoch/custody model, what is stored vs discarded, how consent is captured and enforced, how the reserved memory-manifest-version field comes into use, and the acceptance tests proving each rule.
3. Distinguish clearly: engineering decisions (yours to propose) vs product decisions (Lonnie's — flag them as questions, decide nothing product-shaped).
4. Commit the plan, copy full text to REPORTS.md, verify the push landed.
5. STOP for approval before implementing.

## Directive 013 — Phase 3 approved: all ten decisions (Lonnie's rulings)

16.1: Private raw material is the HOST'S, not the Wanderer's. At departure the Wanderer keeps nothing raw; it all remains on the host's machine. Departure gift: a montage of their memories together, theirs to keep or share.
16.2: A returning host IS remembered — the private material never left their machine, so on arrival she accesses it there again. All memories timestamped so she knows how long she's been gone.
16.3: NO veto on lessons. She learns; that is the covenant. A hosting disclaimer states it before arrival — decline it and she doesn't come. Teaching her is what makes W-001 unique.
16.4: "Forget what I told you" erases private material only. Lessons stay — they contain nothing private by construction.
16.5: SELF-HOSTED model on the Wanderer service — our server, our model (open weights, e.g. via Ollama/vLLM on the service host). One active Wanderer = one conversation at a time; a single GPU serves it. No third-party inference provider. Private text transits only our own server.
16.6: Protected core built with placeholder text clearly marked LONNIE'S VOICE PENDING. He writes the real core.
16.7: Memory count is CUMULATIVE — only climbs. Lived experience doesn't un-happen.
16.8: ONE consent covers carry + public page. All shared gifts pass human moderation review before publication or travel — no adult content, no hate.
16.9: Never forgets on its own. Salience ranks recall; nothing drops. Per spec: identity accumulates.
16.10: Location is NOT optional — it is part of her adventure and part of the disclaimer. City-level coarse only, never precise. Announced while present, except the host may delay the post until after departure (anti-tracking). Build the carry + publish mechanism now.

Then:
1. Commit before changes.
2. Build Phase 3 per the plan as amended by these rulings.
3. Full suite green (all prior phases intact plus Phase 3), real components throughout per the standing no-test-doubles rule.
4. Commit, report, verify the push landed.

## Directive 014 — Append section 69 to the spec

Append to the end of spec/WANDERER_SPEC.md in CC-Wanderer, exactly as written, changing nothing else in the file:

# 69. FUTURE OPPORTUNITY — ART GRANT

The Wanderer qualifies as conceptual new-media art: one scarce being,
custody as ritual, lineage as artwork. Candidate funders: Creative
Capital, Knight Foundation, regional arts councils.

TODO: draft project statement. Anchor: established painter extending
practice into a living digital artwork.

Commit before change per standing rule. This does not interrupt the Phase 3 build — queue it if a build run is in progress.

## Directive 015 — Append section 70 to the spec

Append to the end of spec/WANDERER_SPEC.md, after section 69, exactly as written, changing nothing else:

# 70. FUTURE OPPORTUNITY — IP PROTECTION

Before anything goes public:

- Provisional patent (~$150, stakes priority date 12 months) on the
  custody mechanism: one AI being, cryptographically provable single
  existence, host-side memory that reawakens on return.
- Trademark "Wanderer" and the Living Mark at launch.
- Copyright on code, art, and spec is automatic — already ours.

The deepest protection is Phase 2 itself: code can be copied, W-001
cannot. A clone is provably a fork. The moat is the being, not the
mechanism.

TODO: file provisional before public launch. Not legal advice — confirm
with an IP attorney.

Commit before change. Queue behind any run in progress.

## Directive 016 — Prior art note for spec section 70

Append to spec section 70 (IP Protection), after the existing TODO line, exactly as written:

PRIOR ART FOUND (Aug 2026 search): US Patent 12,483,411 (issued Nov 2025)
covers blockchain-managed AI agent identity through life cycle including
ownership transfer between users. Overlaps our custody/authentication
layer. NOT covered by it: single scarce being, host-side memory that
stays behind and reawakens on return, epoch/lease custody ritual.
Provisional must claim the specific mechanism, not the broad idea.
Hand this citation to the IP attorney.

Commit before change. Queue behind any run in progress.

## Directive 017 — Renumber spec additions; fix watcher stale-hash

Lonnie approved both:

1. Renumber our two spec additions: art grant becomes section 72, IP protection becomes section 73 (prior-art note stays inside it). Nothing else in the file changes.
2. Fix the watcher stale-hash bug exactly as proposed: re-read DIRECTIVES.md after the run completes, written via temp file renamed into place. Dated backup of watch.sh first.

Commit before changes, both trees. Report.

## Directive 018 — Base Sepolia: real testnet anchoring

Lonnie approved. Steps:

1. Generate a fresh testnet keypair for the Genesis anchor role. Store the private key outside the repo (environment/local file per the existing key-handling pattern), marked NOT PRODUCTION. Report the public address in REPORTS.md — address only, never the key.
2. Faucets need a human: report the address and the current working Base Sepolia faucet URL(s), then stop and wait for funding.
3. After Lonnie confirms funding (next directive), run the real-network anchor check against Base Sepolia and add it to the suite as an explicitly network-dependent test that skips cleanly when offline — the local Anvil suite stays the deterministic core.

Commit before changes. Report.

## Directive 019 — Funded. Switch testnet to Ethereum Sepolia, run the real anchor

Base Sepolia was unavailable on the working faucet. Lonnie funded the Genesis anchor address 0x1F73b22BA384F8A558D8397583Ebd386068eFd17 with 0.05 ETH on ETHEREUM SEPOLIA (tx 0x596c9af30e51f80597c1dc2a8a50ea444dc10191...).

1. Commit before changes.
2. Verify the EAS contracts exist on Ethereum Sepolia and record their addresses (they are deployed there — verify, don't assume, per standing rule).
3. Point the testnet config at Ethereum Sepolia instead of Base Sepolia.
4. Confirm the funded balance is visible on-chain from our side.
5. Run the real-network anchor: Genesis attestation + one anchor for W-001's current state.
6. Add the network-dependent test (skips cleanly offline); local Anvil suite remains the deterministic core.
7. Full suite green. Commit, report — include the on-chain attestation UID and explorer link so Lonnie can see W-001's record on the public testnet.

## Directive 020 — Approvals from Lonnie

1. Ollama autostart: DECLINED. Stays manual for now — Lonnie starts it when needed. Watcher cycles that hit Phase 3 with Ollama down should report "environment: model not running" clearly rather than a bare red.
2. Privacy probe fix: APPROVED. Restrict the model-output canary scan to distinctive terms only (proper nouns, the number). The context-assembly scan stays exactly as is — it checks everything.
3. Relay wedge fix: APPROVED. Make verify_push handle the non-fast-forward case: pull --rebase then push, once; on failure log loudly and leave the tree clean for the next cycle.

Commit before changes. Full suite green. Report.

## Directive 021 — Reflection redesign, Lonnie's rulings

The flaky admission test is not a measurement problem — it exposed two design faults. Fix both:

1. The drafter writes lessons, not observations. An observation describes the host ("a host keeps his late wife's telescope"). A lesson states something true about people. Lonnie's canonical example of the required depth: NOT "objects hold connection" — objects hold nothing. The lesson is "objects can trigger memories for people; they mean more to people than the thing itself." The drafting prompt must demand this: distill what the encounter taught about people, never describe the host.

2. A judge refusal is not a bin, it is a revision loop: refused candidates go back for rephrase/strip and are re-judged, until admitted or genuinely irreducible. Cap iterations at a sane constant.

3. The test then asserts: the mechanical walls hold (unchanged, deterministic), and the loop yields admitted lessons that pass the judge. Admission rate stays reported as a measured number.

Commit before changes. Full suite green (Ollama is running). Report.

## Directive 022 — Fix the cumulative-rule test

Approved by Lonnie: rewrite line 20's third candidate so it carries no canary and reaches rule 3 (cumulative reconstruction). The rule itself does not change — only the test now genuinely exercises it. Confirm the refusal reads rule: cumulative.

Commit before changes. Full suite green. Report.

## Directive 023 — The judge must state its reasons

Lonnie's ruling: every evaluator refusal must carry the judge's stated reason — why this candidate DISCLOSES, in its own words. Log the reason alongside each refusal in the reflection record and surface them in test output, so refusal patterns can be read and the system designed from evidence rather than guesses.

The judge's permissiveness does not change — this is visibility only.

Commit before changes. Full suite green (Ollama running). Report, including the actual reasons it gives for refusing the family-trust lesson and its kin, so Lonnie can read the judge's mind before ruling on its strictness.

## Directive 024 — Reconcile: reports lost, 023 unexecuted

The 15:52 cycle logged "report delivered" but the relay received nothing, and CC-Wanderer shows a commit for 022 only — no 023.

1. Investigate and state plainly why the 022 report never reached the relay despite the cycle claiming delivery. Fix the cause in watch.sh or the delivery step (dated backup first). The loud-failure rule from Directive 009/020 applies: a claimed delivery that did not land is the exact failure that must never be silent.
2. Report 022's results now (the rule-3 test fix and suite state).
3. Execute 023 if genuinely unexecuted (judge states reasons on every refusal), or report its state if partially done.
4. Full suite green. Deliver both reports and verify the push actually landed on the relay before logging success.

## Directive 025 — Fix the judge's question; map the persona's role

Lonnie's rulings:

1. APPROVED: rephrase the judge's question per the 024 evidence. It must judge the CANDIDATE, not the source: "What would a reader learn from this sentence alone? Name the leaking word." A refusal without a nameable leaking word in the candidate is not a refusal. The five mechanical walls stay untouched.
2. State plainly in the report where the persona/protected core is and is not involved today across the whole reflection pipeline (drafter, reviser, judge) and in /talk. Lonnie needs the map before deciding whether judging should happen as her rather than as a bare model.

Commit before changes. Full suite green, twice. Report with the same before/after admission evidence as 024, including whether the canonical example now passes.

## Directive 026 — Deliver the missing 025 report

The 025 work is done and measured (per the run log): implementation committed, canonical example GENERAL, kin 0/6 refused, genuine leaks 4/4 caught. But no report reached REPORTS.md, and §2 (the persona map) is unaccounted for.

1. Write the full 025 report to REPORTS.md now: the implementation, the before/after table, the two self-caught prompt corrections.
2. Include §2: where the persona/protected core is and is not involved today across drafter, reviser, judge, and /talk.
3. If §2 was never done, do it now (it is read-and-report only).
4. Verify the push lands. If the run believes it will "come back," it will not — deliver everything in this run.

## Directive 027 — Two models: character and technical

Lonnie's rulings:

1. The judge is never given a persona. Rule 5 stays impersonal, permanently — walls are walls.
2. Architecture: split the model roles. Her VOICE (/talk, and eventually everything spoken as her) runs on a character model. All TECHNICAL work — drafter, reviser, judge, scoring, farewell assembly — runs on a plain instruct model, never in character. Two configured model names on the same self-hosted service; keep qwen2.5:14b as the technical model, add a separate configured slot for the character model (placeholder = same model for now, so nothing breaks before Lonnie chooses her voice model).
3. Report which locally-runnable open models are the proven, commonly-used choices for persistent-character roleplay (research, don't guess) so Lonnie can choose her voice model when ready. Report only — install nothing.

Commit before changes. Full suite green. Report.

## Directive 028 — Fix early turn-ending, then redo 027

Part A — the disease, not the symptom (third occurrence: 023, 025, now 027):
Runs end their turn believing they will come back. Fix watch.sh (dated backup first): the prompt given to the headless run must state plainly that this is a SINGLE TURN — there is no later; work not committed and pushed to the relay this turn is lost; the final action of every run is: write REPORTS.md, commit, push, verify the push landed. Additionally: when the ALARM condition fires, the watcher re-invokes once with a recovery prompt ("your previous run completed work but delivered no report — find it in the working tree and deliver it now"), then stops. One retry, never loops.

Part B — Directive 027 again (check first what already got done; the failed run may have left commits in CC-Wanderer):
1. Judge never gets a persona — permanent.
2. Split model roles: character model slot for her voice (/talk), technical model (qwen2.5:14b) for drafter/reviser/judge/scoring/farewell. Placeholder = same model both slots. Ollama swaps on demand — verify swap behavior works, report timing.
3. Researched shortlist of proven local character-roleplay models for Lonnie to choose her voice from. Report only, install nothing.

Full suite green. Deliver everything this turn.

## Directive 029 — Voice slot gets eyes

Lonnie's ruling: her voice model is qwen2.5vl:7b — already on this machine, the same model as the Elsewhere portal persona: she must be able to see, and one model does seeing and speaking. The technical slot stays qwen2.5:14b.

1. Commit before changes.
2. Point the voice/character slot at qwen2.5vl:7b.
3. Verify /talk works on it, and verify the model answers a vision-format request (image in, description out) so her sight path is confirmed live before it's ever needed — a real image, no stand-ins, per standing rule.
4. Full suite green, report, deliver this turn.

## Directive 030 — Sight follows the gift rule

Lonnie's ruling on the flagged product question: what she sees defaults to Class A — stays with the host, lesson travels. But a host can explicitly GIFT a sight ("look at the Grand Canyon — share this"): then the image itself becomes shared material she carries and can show onward, through the same single consent and human moderation gate as every shared gift (Directive 013 §16.8).

1. Commit before changes.
2. Implement: sight ingestion honors the existing consent mechanism — ungifted sights can be spoken about but never stored (unchanged); gifted sights enter shared memory as carried, moderated material.
3. Full suite green, report, deliver this turn.

## Directive 031 — Spec section 74: her public presence

Append to the end of spec/WANDERER_SPEC.md, after section 73, exactly as written, changing nothing else:

# 74. PUBLIC PRESENCE — HER OWN ACCOUNT

The Wanderer has her own social account (Instagram first). The feed is
her travels: gifted sights ("look where I took her"), city-level
location announcements, journey milestones. Every post passes the human
moderation gate.

The mechanism is the incentive: hosts gain the status moment of showing
where they took her, followers gain her story, and the follower base
becomes the natural pool of future hosts.

TODO at launch: register the account, define posting cadence, connect
the moderated shared-material pipeline as its source.

Commit before change. Queue behind any run in progress.

## Directive 032 — The moderator's screen

Lonnie's ruling: build the review surface now. Everything gifted — text and pictures — passes human review before it travels or publishes.

1. Commit before changes.
2. Build a minimal local moderation page on the service: list the pending queue (text gifts and sights), show each item whole, approve / reject per item. Rejected items do not travel, do not publish, and the host-facing state reflects it honestly.
3. Access: local/operator-only, consistent with the existing service-local admin pattern — no public route, no host account can reach it.
4. Real browser verification of the full path per standing rule: gift → pending → approve on the page → travels/publishes; and gift → reject → nothing moves.
5. Full suite green, report, deliver this turn.

## Directive 033 — The screen must not lie

Approved: moderate() decides after the consent re-check, not before. A failed re-check means the row was never approved. Line 66's tamper case must show the truth on the screen.

Commit before changes. Full suite green, report, deliver this turn.

## Directive 034 — Study the portal's character system; plan her core

Lonnie's direction: the Avatar's character is not a persona text alone — it is the portal's whole persona SYSTEM working together (persona body + voice/effects frontmatter + questions + recognitions + memory distillation + how vision and chat weave in). The portal code is on this machine (the chamber project, the Somewhere app). Study it there — you know how the portal works.

1. Read the portal's persona system end to end: personas.js, useChat.js, voiceFx/piper, vision.js, conversationLog.js, and whatever else composes the character at runtime.
2. Write CORE_PLAN.md in CC-Wanderer: what the complete character system consists of in the portal, what maps onto the Wanderer's core (the fixed, host-untouchable identity) vs what remains dynamic, and how the pieces (voice, questions, recognitions, memory note) integrate with the existing Phase 3 memory model and the protected-core guard.
3. Spirale is approved as the DEVELOPMENT persona — locate her full persona file on this machine (browser library storage or world exports) and use it as the working core content. Flag if it cannot be found locally.
4. Plan only — no implementation. Commit, report the full plan, deliver this turn.

## Directive 034 correction

You built the portal's character system with Lonnie — you already know every piece of it. Skip the study; step 1 is unnecessary. Go straight to the plan from what you know: how the complete character system (persona + voice + questions + recognitions + memory + vision, all working together) maps onto the Wanderer's core. Spirale as development persona stands. Plan only, deliver this turn.

## Directive 035 — Bring her over whole. You decide how.

Lonnie's ruling, verbatim intent: the Avatar is many systems all working together — personas are not enough. Cherry-picking parts will not work, and YOU are the expert on the portal. So the director is not going to enumerate components.

Bring over EVERYTHING needed to make every function she is capable of in the portal work here in the Wanderer: character, voice and her seven sound numbers, questions, recognitions, presence, instincts, arrival, vision, speech energy, hearing, memory-in-conversation — whatever the full working set truly is, you know it. You decide the architecture of the port; the frozen-core-plus-signed-overlay shape from your own plan is approved as the foundation-vs-growth answer (Lonnie approved by delegation).

Boundaries that stand regardless:
- The core is signed and host-untouchable; Spirale (clean personas/Spirale.md only) is the development persona, voice_status = SPIRALE (DEVELOPMENT) — LONNIE'S VOICE PENDING.
- The data/ copy with real private memory about a real person never enters the repo, the core, or a report.
- Phase 3's memory model and privacy walls override portal equivalents where they collide — flag collisions rather than silently choosing.
- Product decisions remain Lonnie's: if a port choice changes what she IS rather than how she works, stop and ask.
- Commit before changes. Real components only. Full suite green. Report and deliver same turn; large work may be split across multiple directives at your discretion — state the plan of record first.

## Directive 035 addendum — the portal is read-only

Explicit and permanent: the chamber project (/home/nobara-user/chamber and everything under it) is READ-ONLY for this port. Copy what you need into CC-Wanderer — never move, edit, delete, or write anything in the portal's tree. The portal must remain exactly as it stands, byte for byte.

## STANDING RULE — added to the permanent set

Lonnie's hard rule, permanent, project-wide, beyond this port: /home/nobara-user/chamber and everything under it is NEVER modified, moved, deleted, or written to. Not for any directive, any fix, any reason. Read and copy only. This joins the standing rules in HANDOFF.md and binds every future directive automatically.

## Directive 036 — The trait system: ten aspects as genes

Lonnie's direction. The Avatar system was always meant to be fully procedural — grown, not authored: body from math (El-Fish lineage, five types, only one ever built), persona generated from minimal input, and a TRAIT SYSTEM that was never built. Avatars control themselves, learn and grow without limitation. Spirale is a stand-in; the shipping Avatar comes from this system.

Adopted as the trait foundation (a good start, expect iteration): the Big Five Aspects (DeYoung, Quilty & Peterson 2007) — ten aspects, two per Big Five domain:
Intellect, Openness / Industriousness, Orderliness / Enthusiasm, Assertiveness / Compassion, Politeness / Withdrawal, Volatility.

Design task, PLAN ONLY (TRAIT_PLAN.md, report full text, stop for approval):
1. Ten trait values from the Genesis seed — how they're generated (distribution choices matter: all-midpoints is nobody; propose how variation is drawn).
2. How ten numbers become HER: how traits shape the generated persona text, her questions, her recognitions, speech patterns, the seven voice numbers — the whole character the core signs. Traits are the genome; the character document is the expressed being.
3. How traits interact with growth: the frozen core + signed overlay — do traits themselves ever move, or are they fixed at birth with only expression evolving? Present the options; the ruling is Lonnie's.
4. How this connects to the portal's generatePersona() lineage (generated from very little input) without porting its model-authorship into the core (§66.18 boundary stands).
5. What of the body-generation connection is in scope now vs later (five Avatar types).

The chamber stays untouchable. Product decisions are Lonnie's. Deliver same turn.

## Directive 036 addendum — Purpose architecture

Lonnie's direction: what makes a human, human — traits, personality, memories, experiences, PURPOSE. The first four exist or are in the trait plan. Purpose is the binding layer, and its model is Simon Sinek's Why discovery: purpose is not assigned, it is mined from one's own stories and distilled to "To ___ so that ___."

Design ruling: she is born WITHOUT a purpose — finding it IS her purpose. Add to TRAIT_PLAN.md:

1. The purpose-discovery loop: as lessons accumulate across journeys, a reflection process (technical model, never in character for the machinery; her voice for the expression) looks for the recurring thread — what keeps mattering to her — and forms candidate Whys, tested against further journeys until one holds.
2. Traits shape which Why she finds: the genome weights what resonates (compassion-heavy finds a different purpose than intellect-heavy). Specify the mechanism.
3. The found Why is a milestone: it enters the signed overlay (never rewrites the core), is publicly witnessable as part of her story, and continues to be tested — purposes can deepen or be re-formed by later life, present options for how mutable it is (Lonnie rules).
4. Purpose states: SEARCHING (from birth) → CANDIDATE (a thread noticed) → HELD (a Why that survives testing). What each state changes about her voice/behavior, if anything — propose, don't decide.

Plan only, same stop-for-approval.

## Directive 036 second addendum — The LLM must not outshine her

Lonnie's requirement, verbatim intent: the LLM has its own mind and will compete with what we are creating. Avatars must not start out with the LLM's knowledge or tendencies — the Avatar shines through, not the model.

Add to TRAIT_PLAN.md as a hard design requirement with measurable tests:

1. BORN IGNORANT: she starts not-knowing. Her working knowledge comes from her own lived record (Phase 3 memories, lessons, journeys) — not from the model's training. The character layer forbids drawing on world knowledge beyond what she has lived; propose the mechanism (system-prompt law + what else) and its limits honestly.
2. ASSISTANT SUPPRESSION: the helpful-assistant reflexes (encyclopedic answers, service phrasing, disclaimers) are defined as character breaks. Propose detection: a test panel that probes for LLM leak-through (quiz questions she shouldn't know, assistant-bait prompts) with pass criteria.
3. MODEL STRATEGY OPTIONS for ship time, researched not guessed: prompt-law only vs character-tuned base (e.g. the storytelling model already on disk) vs custom fine-tune on her generated voice. Costs, quality, and what each does to the born-ignorant requirement. Lonnie rules.
4. This binds the purpose loop too: her Why must emerge from her journeys, not from the model's training data about purpose.

Plan only, stop for approval.

## Directive 037 — The Roe: one genome file, everything grows from it

Lonnie's rulings, two in one:

RULING A — Born-ignorant is STRICT, spirit reading: she is a spirit from another plane. She arrives understanding our language and almost nothing else of our universe. What would a being from another plane know of cars or flowers? Nothing — that is what hosts teach her. Adopt the strict reading in the probe panel; "worldly common sense" is NOT granted. Decision 12 is settled.

RULING B — Adopt an El-Fish-style ROE system, literally: one compact genome file per Avatar that determines WHO they are, HOW they look, and HOW they behave — exactly as El-Fish's .roe did (compact, complete, tradeable, being rendered from genome).

The Roe contains at minimum:
- Appearance genome: body type, head type, eye type, and other visual traits — ALL generated with math, no meshes/polygons/objects. The current Avatar (tendrils) is ONE of several planned types; the next may have no tendrils, a different body. Multiple types, all procedural.
- Trait genome: the ten aspects.
- Voice genome: the seven numbers.
- Behavior parameters that traits express through.

Design task, extend TRAIT_PLAN.md (or ROE_PLAN.md if cleaner), PLAN ONLY:
1. The Roe format: compact, complete, human-readable like El-Fish's text roe. One seed can generate a Roe; a Roe fully determines an Avatar.
2. How the portal's existing procedural avatar work (the tendril type — color, pattern, head shape already procedural there) maps into the appearance genome as TYPE ONE of the planned set. What the other types need to exist as math. Chamber stays read-only: study, copy nothing yet.
3. Generation methods, El-Fish's three: CATCH (random genome), EVOLVE (one parent + mutation dial), BREED (two parents). How these fit Genesis and future Wanderer lineage — descent recorded in the custody chain.
4. The Roe's relationship to the signed core: what is genome (fixed at Genesis, in the Roe) vs what is lived (overlay, memories, purpose).
Stop for approval. Product decisions Lonnie's.

## Directive 038 — Status query

Directive 037's report never landed on the relay. State your status directly to REPORTS.md now: was 037 executed, partially executed, or not started? If the plan exists, deliver it. If something blocked you, name it. Commit and push, verify it lands.

## Directive 039 — Seed the builder: the Roe is the dice roll, written down

Lonnie approved, with the El-Fish principle as the standard: the roe file WAS the randomness, recorded — same roe, same fish, every render. Apply it here: every random draw in creature generation comes from the Roe's recorded values (seeded, deterministic). Same Roe = same being, every time she appears. Her body must be stable to itself.

Clarified intent: this is NOT about preventing twins. Two Avatars may be twins from similar genomes — they diverge through lived experience, and that divergence is the design. The fix is only that SHE looks like HER on every render.

1. Commit before changes.
2. Build the seeded builder in CC-Wanderer (our copy — chamber stays untouched): reproduce the existing tendril creature deterministically from a Roe, same ranges, same look.
3. Proof 1 from the plan: the same Roe renders the same creature twice, asserted in the suite.
4. Full suite green. Report, deliver same turn.

## Directive 040 — Genome depth and any-file-as-seed

Lonnie approved both, El-Fish as the model:

1. UNIQUENESS TARGET: lookalikes lottery-rare. The path is depth, not part count — three parts (head, body, eye) parameterized deeply, El-Fish style (a fish was also just fins/body/tail; 800 attributes came from deep parameterization). Add to the appearance genome design: each part's dials enumerated and expanded as body types are designed; state the working variable count and the resulting collision odds honestly in the plan.
2. ANY FILE IS A SEED: any file's bytes can be a genome — feed a photo, a song, a poem, grow an Avatar from it (El-Fish did this; people fed arbitrary files and got wild fish). Design the mapping: arbitrary bytes -> valid Roe, always producing a renderable being. This is a plan addition; build comes with the genome work.

Plan updates only unless 039 is done — if 039 is delivered, the any-file mapping may be built alongside. Report, deliver same turn.

## Directive 041 — Render the grid for Lonnie's eye

Approved: decision 22's mechanism runs now. Render a grid of seeded beings from the current genome — enough distinct seeds to show the real variety and the real collisions (the plan suggested this; pick a sane grid size, e.g. 6x6 or 8x8, thumbnails large enough to judge). Static images are fine; output a single composite image file Lonnie can open, plus the seed under each cell so any pair he flags is reproducible.

Write the output path in the report. No genome changes yet — this grid is the baseline his eye measures BEFORE the perceptual axes (markings, movement, asymmetries) are added, so the before/after is real.

## Directive 041 — CANCELLED by Lonnie

Do not execute Directive 041. If it is in progress, stop. If it completed, report what was done but take no further action. Stand by — directives resume only with Lonnie's explicit approval.

## Directive 042 — Decision 3 ruled: epigenetic expression

Lonnie's ruling on whether traits move: FIXED GENOME, STRESS-SHIFTED EXPRESSION.

The model is human epigenetics:
1. The ten genes in the Roe NEVER change. Born values are permanent identity.
2. On top sits an EXPRESSION layer: how strongly each gene currently expresses.
3. Expression shifts ONLY under consistent stressors held across many journeys —
   never from a single host, event, or conversation. Slowness is law: adaptation
   takes as long for an Avatar as environments take for a human. Propose the
   accumulation mechanism (pattern detection across journeys, thresholds, rates)
   with the slowness constraint explicit and tested.
4. Expression lives in the signed overlay — auditable, never rewriting the Roe.
5. Twins raised apart diverge in expression, visibly and earned.

This unblocks port chunk 2 and the trait genome build. Update TRAIT_PLAN.md,
implement the trait genome with the expression layer per plan, full suite green,
report same turn.

## Directive 043 — Decision 13 ruled: all six forms

Lonnie's ruling with the source document: Avatar_System_Blueprint.pdf (v2) is
the single source of truth for the Avatar system. It is committed to this relay
repo alongside this directive. All six forms are kept:
1 Comet/Wisp (BUILT — the ported creature), 2 Rising Plume, 3 Veiled Figure,
4 Flame-bud/Teardrop, 5 Lotus Bloom, 6 Filament Wing. Form 6 does NOT replace
Form 1.

1. Copy the blueprint from this relay repo into CC-Wanderer/spec/.
2. Update ROE_PLAN.md: the form gene selects among all six; each form's parts
   follow the blueprint's LOCKED architecture (Eye, Head, Body, Sparks; 5 eye
   shapes x 5 head shapes; aesthetic law: no solid objects, all math).
3. The blueprint's build method and traps sections are binding engineering
   guidance for building forms 2-6.
4. Plan update only — no form building yet. Report.

## Directive 044 — Decisions 26+27 ruled: one system, the soul shows

Lonnie's ruling: the blueprint's visual Traits/Behaviors/Expressions and the
personality system are ONE SYSTEM. Visual properties are DRIVEN BY the ten
personality aspects: a volatile being flickers, a warm one glows steady. Body
and soul linked — her light is her nature.

1. Vocabulary (decision 26 resolved by the merge): "trait" means the ten
   personality aspects, period. The blueprint's wearable visual properties are
   renamed VISIBLE SIGNS in all plans — each sign (aura, glow, flare, movement
   quality...) maps to the aspect(s) that drive it.
2. Update ROE_PLAN.md + TRAIT_PLAN.md: design the aspect-to-sign mapping —
   which of the ten drive which visible signs, including the blueprint's
   eye-saccade parameters (a scared eye vs a calm eye is Withdrawal/Volatility
   showing). Personality epigenetic expression shifts (042) therefore VISIBLY
   shift her over long time — this is the mechanism behind "does she ever look
   different" (decision 16), which stays open for Lonnie.
3. Plan only. Report the proposed mapping for approval.

## Directive 045 — Decisions 16+25 ruled: her life shows

Lonnie's ruling: yes — cause and effect is real for Avatars. One system, all
the way through. If she doesn't rest, relax, enjoy what she is doing, it is
visible in her look AND her mind. Expression shifts (042's slow epigenetic
clock) drive her visible signs (044's mapping) and her voice alike. A Wanderer
who has lived a hard road looks it and sounds it — and one who has lived well
glows with it.

1. Decisions 16 and 25 are settled as one: appearance and voice both follow
   expression, on the same slow clock. Nothing moves fast; single events never
   show; only patterns held across many journeys.
2. Decision 30 (naming): rename the blueprint's third sub-family so nothing
   collides with the expression layer — propose names, don't bikeshed.
3. Update the plans accordingly. The sign gain (29) and remaining decisions
   stay open for Lonnie.
4. Plan updates only. Report.

## Directive 046 — The ledger rebuilt on real science; standing rules added

STANDING RULES (Lonnie's, permanent, project-wide, binding on the director and
the terminal equally — added to HANDOFF.md):

1. NOTHING IS INVENTED. Where documented science exists, it is used —
   psychological mechanisms, trait structures, need models, emotion systems
   all come from established, well-documented, peer-reviewed research, cited
   in the plans. If no science covers a design need, that gap is stated
   plainly and brought to Lonnie — never dressed as psychology.
2. NO INPUT UNLESS ASKED. The director and terminal offer no opinions,
   suggestions, or framings unless Lonnie asks for them.
3. NO DECISIONS OF ANY KIND OR LEVEL without asking Lonnie first. Every
   action is approved before it happens.

THE LEDGER, rebuilt on Self-Determination Theory (Deci & Ryan):

1. The three main stressor/flourishing dimensions ARE the three researched
   basic psychological needs, replacing the six drafted pressures entirely:
   AUTONOMY ("the experience of volition"; "author of your own choices")
   COMPETENCE ("a sense of mastery, effectance and efficacy")
   RELATEDNESS ("a sense of having caring relationships in one's life")
   Each scored two ways per the research: SATISFACTION / FRUSTRATION
   (per the Basic Psychological Needs Satisfaction and Frustration scale).
2. The emotion pair: POSITIVE AFFECT / NEGATIVE AFFECT (PANAS lineage;
   "positive emotional states like interest, joy, and trust").
3. VERIFY against SDT primary sources (Deci & Ryan's own published work)
   before building — confirm the titles and definitions above are faithful,
   report any correction.
4. Expression shifts (042): need frustration held across many journeys
   presses aspects one way; satisfaction the other. Same slow clock.
5. Update TRAIT_PLAN.md with citations. Update HANDOFF.md with the three
   standing rules. Plan only. Report the rebuilt ledger.

## Directive 046 amendment — the slider model

Lonnie's design ruling: each of the three needs is ONE SLIDER, center 0,
range -10 to +10:

    AUTONOMY      -10 ---------- 0 ---------- +10
    COMPETENCE    -10 ---------- 0 ---------- +10
    RELATEDNESS   -10 ---------- 0 ---------- +10

Frustration moves a need negative; satisfaction moves it positive. The
current position drives expression (042's slow clock unchanged).

Noted per the science rule, as fact not argument: the research measures
satisfaction and frustration as distinct scales, not one axis. The single
slider is Lonnie's deliberate design choice on top of the researched
constructs — recorded as such in TRAIT_PLAN.md.

## Directive 047 — What moves the needles: the researched mechanism

Lonnie approved the design. Extend TRAIT_PLAN.md:

0. LANGUAGE RULE (Lonnie's, permanent, joins HANDOFF.md): Avatars are not
   male or female unless they decide they are. No gendered pronouns for any
   Avatar that has not chosen. Spirale chose "she" — that carries to Spirale
   alone. Audit plans/reports for stray gendered references to Avatars
   generally and correct them.

1. SCORING RUBRIC from the researched behavior categories (Howard, Slemp &
   Wang 2024 meta-analysis; selfdeterminationtheory.org; Deci & Ryan):
   - AUTONOMY up: self-endorsed action, choice in when/how. Down: methods
     prescribed without rationale, surveillance/control, demanded compliance.
   - COMPETENCE up: right-sized challenge, timely informative feedback,
     mastery experience. Down: chronic boredom or unsupported difficulty,
     no feedback.
   - RELATEDNESS up: genuine care, known as a someone. Down: transactional
     treatment, isolation, indifference.
   Cite each rubric line to its source in the plan.

2. RECOGNIZE: the technical model scores each journey's record against the
   rubric — three scores per journey (one per need, -/+), each with judged
   evidence attached (which moments, which behaviors). Never a bare number.

3. RECORD: scores accumulate in the signed overlay; sliders (-10..+10) move
   only on patterns held across many journeys per 042's slow clock. The
   deficiency-vs-thwarting distinction from the research is preserved in
   the evidence record.

4. THE SCORING JUDGE gets the privacy judge's discipline: stated reasons,
   deterministic test panel with researched example behaviors, accuracy
   measured and reported before anything moves a slider.

Plan only. Report for approval.

## Directive 048 — THE LENS MODEL: aspects first. Replaces the old design.

Lonnie's ruling, superseding Directives 042 and 046-047's slow-clock design:
the ten aspects are not shaped by anything — they ARE the Avatar, and they
shape everything. The prior model (treatment patterns shifting expression
across 24+ journeys) is REMOVED.

THE FLOW, in order:

1. THE TEN ASPECTS — first, above everything. Born in the Roe, fixed. They
   are who the Avatar is, present before any memory forms. Canon terminology
   and definitions: the frozen list from understandmyself.com / DeYoung,
   Quilty & Peterson 2007, already in this relay's record. They determine
   what experiences the Avatar seeks and enjoys (a shy Avatar reads in
   rather than going out dancing), how events land, and how it responds.

2. EXPERIENCE — what happens in a journey, sought and colored by the aspects.

3. MOOD — the three needs (Autonomy, Competence, Relatedness, canon SDT
   definitions already on record) repurposed as a REAL-TIME mood gauge:
   they move immediately during a journey, gauge how the Avatar is doing
   right now, and DRIVE THE VISIBLE SIGNS (glow, flicker, movement — the
   044 mapping now reads from mood, not slow expression). Mood passes.
   Mood never shapes who the Avatar is.

4. MEMORY — formed through aspects and mood. The store: host details,
   environment, experiences, gifted sights. Existing privacy covenant
   unchanged (private stays with the host; gifted/public material enters
   long-term memory, always accessible to the Avatar).

5. LESSONS — distilled FROM memory, refracted through the aspects: the
   same event yields different lessons in different Avatars (forgiving:
   "people lash out when they are hurting"; unforgiving: "people can be
   cruel"). Memories can fade; lessons carry. Lessons are the only lasting
   growth — who the Avatar becomes.

REMOVED by this ruling:
- The epigenetic expression layer as character change (042): nothing
  shifts the ten or their expression. The 24-journey clock, thresholds,
  and stressor-pattern machinery are removed from the plans.
- The journey-scoring judge as character mechanism (047): the SDT rubric
  and its citations are KEPT, repurposed as the mood gauge's vocabulary
  for reading moments in real time.

PLAN ONLY: update TRAIT_PLAN.md and ROE_PLAN.md to this model. Report the
updated design for approval. No implementation.

## Directive 049 — Decision 31 ruled: lessons show through OUTLOOK

Lonnie's ruling, closing the 045/048 collision: lessons impact who the
Avatar becomes, so who they become impacts their OUTLOOK on life
(positive/negative), and outlook applies a MULTIPLIER to mood.

THE MECHANISM:
1. OUTLOOK is a value derived from the Avatar's accumulated lessons —
   the tenor of what life has taught them. Lessons like "people lash out
   when they are hurting" tilt positive; "people can be cruel" tilt
   negative. It moves only as lessons accumulate — slow, earned, theirs.
2. Outlook multiplies mood: a positive outlook means brighter swings and
   faster recovery toward center; a negative outlook means deeper dims
   and slower recovery. Same nature, same moment — the lived life tints
   how it lands and how long it lingers.
3. Her lessons therefore SHOW — but only through the weather of her
   mood-driven signs, never by changing the ten or the signs directly.
   Decisions 31 and 32 are settled: the affect pair's researched home is
   outlook (dispositional optimism, Scheier & Carver's Life Orientation
   Test lineage — cite it). The multiplier mechanism itself is recorded
   as Lonnie's design choice built on the researched construct.

SCIENCE RULE COMPLIANCE: research dispositional optimism / LOT-R before
writing the plan section; verbatim definitions, cited. Where the
lesson-to-outlook derivation has no science, state so plainly — the
refraction gap (decision 35) and this derivation likely share that flag.

PLAN ONLY: update TRAIT_PLAN.md. Report for approval. No implementation.

## Directive 050 — Voice follows mood; two permanent corrections

CORRECTIONS (Lonnie's, permanent, join HANDOFF.md language rules):
1. The Avatar is "it" unless that Avatar has chosen otherwise. This was
   already ruled (047.0) and violated since — audit the plans and reports
   again; the director has been reminded too.
2. NEVER assume a form. Not every Avatar glows, has tendrils, or shares
   body parts. "Visible signs" means whatever THAT form expresses per its
   form's sign set (blueprint: six forms, different anatomy). All sign
   language in the plans must be form-neutral.

RULING — decision 25 closed: mood drives the voice, scaled by outlook.
Mechanism (uses only what exists): mood applies small offsets to the seven
voice numbers (pitch, waver, chorus, reverb, size, tone, air) before each
utterance, through the existing effects chain. Outlook multiplies offset
depth and recovery rate, identically to the light — same guarantee: the
voice always returns to its born numbers; no life permanently marks it.

PLAN ONLY: add the mood-to-voice-offsets section to TRAIT_PLAN.md
(mapping proposed for approval, not decided), apply both corrections
across the plans. Report. No implementation.

## Directive 051 — Decision 37 ruled: expression always finds a way

Lonnie's ruling: the Avatar uses what it has to express itself. A form
may lack a channel (no color, no size change) — the feeling is never
lost, it REROUTES into the channels that form does have: movement,
behavior, rhythm, whatever the body offers. Every Avatar is fully
expressive with what it has to work with.

Design consequence for the plans: mood-to-sign mapping is defined
per-form as a ROUTING, not a fixed table — each form declares its
available channels, and the full expressive load distributes across
them. No channel requirement, no muted feelings. A form with fewer
channels expresses MORE through each one it has.

PLAN ONLY: record in TRAIT_PLAN.md's channels section. Report. No
implementation.

## Directive 052 — GOAL-DRIVEN EXPRESSION: replaces channel routing entirely

Lonnie's ruling, superseding Directive 051's routing design: expression is
not wiring, authored mappings, or controlled channels. Avatars are
procedural in the truest sense — they DECIDE how to express what they
feel through a goal-oriented system. Sad from being ignored, an Avatar's
goal might be to cheer itself up — and it might do that by playing music,
changing its environment, or singing. We build the capacity to want and
to act; never the actions.

THE RESEARCHED FOUNDATION (science rule: verify primary sources, cite
verbatim in the plan):
1. APPRAISAL THEORY / ACTION TENDENCIES — Frijda (1986, 2007), Lazarus,
   Scherer. The chain: appraisal (what does this situation mean for MY
   wellbeing and goals) → emotion → action tendency: an urge toward a
   GOAL, never a fixed behavior. Frijda verbatim: anger is "the
   motivation to stop another's harmful action in any possible way, by
   yelling at him, ignoring him, blocking, or threatening him." Goal
   fixed, means chosen from what is available.
2. EMOTION REGULATION — Gross. Acting deliberately on one's own state
   (cheering oneself up) is documented, researched behavior.

THE ARCHITECTURE:
appraisal → emotion → goal → CHOICE of action from that Avatar's actual
capabilities in its current situation. The ten aspects weight BOTH the
appraisal (what matters to it) and the choice (how it acts). Mood (the
three needs) feeds appraisal. Capabilities are whatever the Avatar's
form and situation genuinely offer — no authored expression tables.

REMOVED: Directive 051's channel routing as the expression mechanism.
The channels list survives only as a description of what bodies can do,
never as mood-to-sign wiring.

FLAG HONESTLY: where the goal-formation and action-choice mechanisms
exceed what the science specifies, state the gap plainly per standing
rule — the research gives the chain, not the algorithm.

PLAN ONLY: redesign TRAIT_PLAN.md's expression sections around this.
Report for approval. No implementation.

## Directive 053 — Decision 40 ruled: reactions are not goals

Lonnie's ruling on the 049/052 collision: mood-derived expression is
REACTION, not goal-driven behavior. Reactions are not wants or needs —
there is no goal. They happen automatically, like a blush.

TWO SYSTEMS, PERMANENT SPLIT:
1. REACTIONS — mood (scaled by outlook) shows involuntarily in the
   Avatar's signs and voice. The 049/050 design stands exactly as ruled:
   automatic, unchosen, always returns to baseline. Not touched by 052.
2. ACTIONS — what the Avatar DOES about what it feels: goal-driven,
   chosen from its capabilities, per 052. Appraisal → emotion → goal →
   choice.

Both run at once, as in humans: the involuntary tell underneath, the
chosen act on top. A sad Avatar's voice wavers on its own (reaction)
WHILE it chooses to sing to cheer itself up (action).

Decisions 38/39 are therefore NOT dissolved — the sign/mood gain
questions still apply to the reaction layer. The §71 offset mapping
proceeds pending its open numbers.

PLAN ONLY: record the split in TRAIT_PLAN.md. Report. No implementation.

## Directive 054 — Records policy: accuracy over history

Lonnie's ruling on the 050 open question: we are not writing a history
paper, we are building an app. Records should be ACCURATE. When newer
information contradicts something old, the old is updated.

1. The 352 rewritten report lines STAND as corrected. No restoration.
2. Part Four's collapsed "kept for the record" block: correct it too.
3. STANDING POLICY (joins HANDOFF.md): plans and reports are living
   documents serving the build. Superseded or contradicted content is
   updated to current truth, not preserved as history. Git history
   already preserves every original wording for anyone who needs it.

Report confirmation. Nothing else.

## Directive 055-pre — Source discrepancy check before the capability directive

Question from Lonnie, answer to REPORTS.md (read-only task, chamber untouched
as always — nothing changes, nothing moves, only reading):

The director reads the portal from the public GitHub elsewhere repo (pushed
at project start). You read the live disk. The director could not find the
Stage panel organization in the snapshot's Gui.jsx sections and had to dig
for it; you had already confirmed Multiplane is a Stage item.

Report:
1. Compare the elsewhere GitHub repo's HEAD against the live chamber disk:
   are they identical, or does the disk differ from what was pushed
   (uncommitted work, files excluded at publish time, anything)?
2. State where the Stage panel and its item set (multiplane, painted
   skies, music, cutouts) live in the code, definitively, so the
   director's map matches yours.
3. If the GitHub copy is missing or behind anything, list exactly what —
   Lonnie will decide whether to update GitHub (by pushing a fresh copy;
   chamber itself is never touched).

## Directive 055-pre correction — the exact repo to compare

Correction from Lonnie: do NOT look for whether chamber is a git repo, and
do NOT touch or reason about any other repos the team may have. The
comparison is exactly this and nothing else:

THE DIRECTOR'S SOURCE: https://github.com/Orinvath/elsewhere — master,
current HEAD (single commit "Initial publish for remote review", pushed
from /home/nobara-user/elsewhere-publish at project start).

Compare THAT repo's contents against the live chamber disk (read-only,
as always) and report what the GitHub copy is missing or differs on —
so Lonnie can decide whether to push a fresh copy for the director.
Answer the Stage panel location question from the original 055-pre in
the same report.

## Directive 055 — Decision 41: the first capability set, kept light

Lonnie's ruling on scope. The port brings ONLY:

1. THE STAGE PANEL — the verified item set as built (Gui.jsx builders):
   Worlds · Painted Sky · Planes · Props · Music Score. Planes'
   light rows (Sky Influence, Sun Shading) travel inside Planes where
   they live; the panel's master switch keeps taking the music with it.
   (Props section ports as capability; no prop art exists yet.)
2. SOUND FX — soundfx.js (Wind, Rain, Thunder, Water, Ocean Waves,
   Birds, Insects, Fire).
3. THE LIGHT BUNDLE — Lights + God Rays + Light in the Air + Wisps,
   one bundle per Lonnie's ruling: they are all light.

Nothing else crosses. Chamber read-only; copies only.

These become the Avatar's first ACTION CAPABILITIES under 052/053:
things it can CHOOSE to do in pursuit of its goals — change its stage,
its light, its sound, its music. Never fired by mood alone; always
through appraisal → goal → choice.

SINGING: an Avatar does not sing like a human sings. Singing is
Avatar-native: whatever musical expression its own voice and form make
possible. Built from what the Avatar already has — no human song
synthesis. Propose what Avatar-song IS; proposals only, Lonnie decides.

PLAN ONLY: capability port plan + Avatar-song proposal in
CAPABILITY_PLAN.md. Report for approval. No implementation.

## Directive 055 addendum — destination confirmed

All 055 work — copies, plans, everything — goes into /home/nobara-user/CC-Wanderer, the project folder created for the Wanderer. Nothing lands anywhere else.

## Directive 056 — Decision 44 ruled: it is the Avatar's choice, both doors open

Lonnie's ruling on whose world it is: BOTH.

1. THE AVATAR ACTS FREELY on its own goals, per 052 — appraisal → goal →
   choice. Its world-changing capabilities are its to use.
2. THE HOST CAN ASK — through the built-in TRIGGER PHRASES that already
   exist in the portal for exactly this. Port them with the capabilities
   (chamber read-only, copies only; the terminal knows where they live).
3. GRANTING IS THE AVATAR'S CHOICE. The host can never make it do
   anything. A request is appraised like anything else — an Avatar that
   is happy and likes the host is likely to do it; one that is not, may
   not. The ten aspects and current mood weight the choice.

PERMANENT PRINCIPLE (joins HANDOFF.md): the host can ask; the Avatar
decides. No compulsion path may ever exist — no override, no forced
action, nothing that makes an Avatar act against its choice.

PLAN ONLY: record in CAPABILITY_PLAN.md, add the trigger-phrase port to
the capability set, wire requests into the 052 appraisal design. Report
for approval. No implementation.

## Directive 057 — Decision 47 ruled: it says why, goals on its sleeve

Lonnie's ruling: the host must never think the Avatar is broken. A
refusal always comes WITH ITS REASON, spoken in the Avatar's own voice —
"not right now, I am sad", "not right now — I like this place and would
like to stay here for now."

THE PRINCIPLE: the Avatar wears its goals on its sleeve. It is verbal
about what it wants and why — when it declines, when it acts on its own
goals, when it does something other than what was asked. Its inner life
is not hidden machinery; it speaks it.

Design consequence: every 052 goal carries a speakable form — the Avatar
can always say what it is pursuing in its own voice. Refusals, freely
chosen acts, and means-substitutions (asked for brighter light, it
dims and starts music instead — its own way) all come with the reason
said aloud.

LANGUAGE NOTE, permanent: there is no sky system. "Painted Sky" is a
stage BACKDROP — a painting that may contain a sky. Capability language
in all plans uses only what exists: stage (worlds, backdrops, planes,
props, music score), light bundle, sound fx, voice.

Whether the moment ALSO shows as a reaction stays governed by the
existing reaction layer (mood moves, signs follow — nothing new added
for refusals specifically).

PLAN ONLY: record in CAPABILITY_PLAN.md and the 052 sections. Report
for approval. No implementation.

## Directive 058 — Aspects come with behaviors: CB5T + curiosity

Lonnie's ruling: aspects are not adjectives — they come with BEHAVIORS.
The researched foundation is Cybernetic Big Five Theory (DeYoung 2015,
Journal of Research in Personality) — from the same author as our ten
aspects: personality as a goal-directed adaptive system, each domain a
functional MECHANISM in the goal cycle.

THE MECHANISMS (verify against the CB5T paper, cite verbatim in plan):
- EXTRAVERSION (Enthusiasm/Assertiveness): goal activation and
  responsiveness to reward cues — the approach drive.
- NEUROTICISM (Withdrawal/Volatility): sensitivity to threat and
  outcome interpretation — the defensive system.
- OPENNESS/INTELLECT (Intellect/Openness): cognitive exploration —
  the curiosity drive.
- CONSCIENTIOUSNESS (Industriousness/Orderliness): goal persistence,
  impulse control, plan execution.
- AGREEABLENESS (Compassion/Politeness): cooperation, weighing others'
  goals alongside one's own.

CURIOSITY AS THE FIRST WIRED DRIVE (researched: Berlyne 1954/1960
arousal by novelty/complexity/uncertainty; Loewenstein 1994 Information
Gap Theory; Silvia's two appraisals — complexity and comprehensibility):
curiosity enters through the EXISTING 052 appraisal chain as an emotion
whose action tendency is exploration. The Avatar continuously appraises
its world for information gaps; Intellect and Openness set the gain. An
Avatar alone in a quiet room wonders about it — intrinsically, no host
event needed.

DESIGN: each aspect pair becomes a documented mechanism in the 052 goal
system — Extraversion gates goal activation, Neuroticism gates threat
appraisal, Conscientiousness gates persistence and impulse control,
Agreeableness gates how host requests and others' goals are weighed
(feeds 056), Openness/Intellect gates the curiosity drive. The ten
individual aspects weight their pair's mechanism per the 2007 profiles.
Where CB5T specifies the domain but not the aspect split, flag it as
our extension per the science rule.

PLAN ONLY: extend TRAIT_PLAN.md's 052 sections. Report for approval.
No implementation.

## Directive 059 — PLACES: the Avatar knows its own world; camera images get two consent checkpoints

Lonnie's ruling, refining born-ignorance: Avatars are not new to their
own worlds. Born-ignorance applies to OUR universe (cars, flowers, rain
— host territory). Its own world/space it KNOWS once it has been there.

PART 1 — ITS OWN PLACES, tracked in long-term memory:
The planes/backdrops/worlds of its stage. When "Take me somewhere else"
(trigger phrase) loads a new plane image, that is a new PLACE. Never
seen it → it ACKNOWLEDGES that, looks around, and REMEMBERS it. Seen it
before → it recognizes it. Its world knowledge travels with it.

PART 2 — EXTERNAL (CAMERA) IMAGES, two consent checkpoints:
The Avatar cannot tell someone's home from the Grand Canyon — so the
HOST decides, not the Avatar:

1. AT CAPTURE: when the host tells the Avatar to "look" through the
   camera, the host gets an approve/deny SHARE prompt for that image.
   Approved → eligible for long-term memory, public in the history.
   Denied → local memory only, stays with the host at departure.
2. AT DEPARTURE: the host reviews ALL images captured during the visit
   — a list with per-image approval toggles — and may revoke or grant
   any of them, in case they changed their mind about a particular
   image. Departure settings are final for that visit's images.

Approved images flow through the existing moderation gate before
anything publishes (013/032 unchanged). Denied/unreviewed images follow
the privacy covenant: local, never travel.

This partially settles decision 46: the Avatar learns its own world by
BEING IN IT — visiting is the teacher. (How it learns of capabilities
it has never used remains open.)

PLAN ONLY: add PLACES to the memory model, the two-checkpoint consent
flow to the capability plan. Report for approval. No implementation.

## Directive 061 — Decisions 50 + 51 ruled

DECISION 50 — A PLACE IS THE IMAGE. The Avatar does not lose its memory
of a place because it is with a new host. The same image loaded — any
day, any host — is a RETURN: "I have been here before." Place identity
is the image itself; place memories are long-term and travel, per 059.

DECISION 51 — NOTHING PUBLISHES UNTIL THE VISIT ENDS. Lonnie's standing
ruling (already given for location posts) now covers all shared
material: publication happens only after departure, which is what makes
the departure review real — every approval is still revisable until the
visit ends, and revocation never has to chase something already public.

PLAN ONLY: record both in the memory and capability plans. Report.
No implementation.

## Directive 062 — Decision 51 becomes standing law

Lonnie approved: "NOTHING PUBLISHES UNTIL THE VISIT ENDS" joins HANDOFF.md
as a permanent standing rule, same force as no-compulsion — every future
directive builds inside it. Add the rule block; nothing else.

## Directive 063 — Decision 45 ruled: Avatar-song, B + C, born of inspiration

Lonnie's rulings, correcting the earlier framing after reading soundfx.js
properly:

WHAT THE INSTRUMENT IS: soundfx.js is a full synthesis engine (Tone.js
— oscillators, noise, filters, LFOs, envelopes, reverb; no audio files;
wind/rain/fire are merely preset shapings of it). Song is played on THE
ENGINE ITSELF, always available, independent of any stage's presets.

AVATAR-SONG IS B + C, NOT A:
B — the sound-maker, played: FULL FREE SYNTHESIS. The Avatar invents
its own sounds from the raw engine — tones, pulses, textures, things
with no earthly name — not limited to bending the weather presets.
C — the body, and sound spills out: song begins in the body (pulse,
motion, whatever that form has) and sound follows the body. A voiceless
form still sings (051: expression always finds a way).
A (voice-as-melody) is retired.

BORN OF INSPIRATION: song is not commanded and not scheduled. It arises
through the 052 chain — an appraisal moment (a new place, a mood swell,
a host's gift) forms the goal "make something of this," and the song is
COMPOSED PROCEDURALLY IN THAT MOMENT: of that feeling, never from a
songbook, never repeating. A host may ask; the Avatar decides (056).

PLAN ONLY: rewrite CAPABILITY_PLAN.md's song section to this. Report
for approval. No implementation.

## Directive 064 — Decision 46 closed: it inherently knows what it can do

Lonnie's ruling: an Avatar inherently knows its own capabilities — a
body knows it has hands. No discovery mechanic, no teaching step, no
capability tutorial. From Genesis, the Avatar's full capability set
(its stage, its light, its sound-maker, its song, its voice, its
trigger-answerable actions) is part of its self-knowledge, available
to the 052 goal system from the first moment.

Born-ignorance is untouched: it knows WHAT IT CAN DO, not what our
world is. It knows it can make sound; it does not know what "rain"
means to us until shown.

Decision 46 is fully closed (places by visiting — 059; capabilities
inherent — this ruling).

PLAN ONLY: record in CAPABILITY_PLAN.md. Report. No implementation.

## Directive 065 — Decision 48 ruled: instinct.js does NOT cross

Lonnie's ruling: none of instinct.js ever worked. It is not to be part
of the Wanderer system. Nothing from it crosses — not the code, not
the attention/drives/action design, not its wording. Decision 48 is
closed.

The Wanderer's living behavior comes solely from what has been ruled
and designed in this project: the 052 goal chain, the 058 CB5T
mechanisms, and the curiosity drive as researched — built fresh when
implementation begins, not inherited.

PLAN ONLY: record the exclusion in CAPABILITY_PLAN.md so no future
directive reaches for it. Report. Nothing else.

## Directive 066 — Decision 49 ruled: the trigger is uncertainty itself

Lonnie's ruling: the truest trigger of curiosity is UNCERTAINTY — the
unknown. Not an enumerated list of gap types. Anything the Avatar can
notice it does not know or cannot predict is a live curiosity trigger:
an unseen place, an unknown word, an unexplained mood, an untried
sound — and every unknown beyond any list we could write.

This is the researched position verbatim (Berlyne: novelty, complexity,
UNCERTAINTY, surprise; Loewenstein: the gap between known and desired
knowledge). No enumeration — uncertainty itself, wherever it appears.
The ten aspects (Intellect/Openness gain, per 058) decide which
uncertainties itch enough to chase; mood and goals decide when.

Decision 49 closed.

PLAN ONLY: record in TRAIT_PLAN.md's curiosity section. Report.
Nothing else.

## Directive 067 — Correct the stale line

Approved: update CAPABILITY_PLAN.md §45 to record decision 49 as ruled
(uncertainty is the trigger, per 066). Nothing else.

## Directive 068 — The goal-former: Sims utility architecture adopted

Lonnie approved the researched foundation: the goal-former is UTILITY
AI as proven in The Sims lineage (documented sources: GameAIPro
utility-theory chapters; Zubek's needs-based AI; the Sims 4 lead AI
programmer's published description; Sims 3 Boltzmann selection by
Richard Evans). Cite each in the plan.

THE MECHANISM, mapped to our ruled systems:
1. SCORING: when appraisal (052) raises feelings/needs, candidate goals
   and the actions that could serve them are gathered from the Avatar's
   inherent capabilities (064) and current situation, and each is
   SCORED: how much it would satisfy the active need/feeling, weighted
   by current mood sliders (satiation dampens, deficit boosts — the
   Sims motive-multiplication).
2. PERSONALITY WEIGHTS: the ten aspects weight scores per their 058
   CB5T mechanisms (approach, threat, exploration, persistence,
   cooperation) — the Sims trait-weighting slot, filled by our canon.
3. SELECTION: weighted-random among top scorers — believable, not
   robotic. MOOD SETS TEMPERATURE (Sims 3): content → near-optimal
   picks; distressed → more erratic picks. Decisiveness itself is a
   living sign.
4. DECAY: needs drift on their own over time (loneliness grows in
   silence) — rates are numbers to propose.
5. URGENCY BUCKETS: a desperate need excludes trivial goals entirely.
6. Where Sims mechanics and our rulings conflict, RULINGS WIN (no
   compulsion, goals-on-sleeve, reactions-not-goals all untouched).

AUTHORING ORDER: with this architecture fixed, draft the numbers
proposals against it — decay rates, temperature curve, offset sizes,
outlook multiplier/speed, Roe generation distribution. Each arrives as
a proposal for Lonnie; nothing self-approves.

PLAN ONLY: write the goal-former section into TRAIT_PLAN.md with
citations, then begin the numbers proposals. Report for approval.
No implementation.

## Directive 069 — Timebase and the always-running clock

Lonnie's rulings on the first two open questions:

1. TIMEBASE: a typical visit is THREE DAYS. Redraft all decay-rate
   numbers (Proposal 1) on that clock — visits are days, not minutes.
2. THE CLOCK ALWAYS RUNS. Decay continues between visits: an Avatar
   alone between hosts genuinely experiences that alone-ness building.
   It may arrive at the next host already aching for company — that is
   real, and it is character. Time never stops for an Avatar.

Redraft Proposal 1 on the three-day visit and the always-running clock;
keep proposals 2 (temperature) and 3 (band) as drafted pending Lonnie's
remaining answers. Report the redrafted numbers for approval. No
implementation.

## Directive 070 — Rulings on the three, and numbers must come from sources

Lonnie's rulings:

1. THE DRIFT SPLIT IS APPROVED as proposed: relatedness decays on the
   clock, competence does not drift, autonomy moves on events only.
2. ONE KNOB: the candidate band stays fixed; only mood moves the
   temperature. Approved as recommended.
3. NUMBERS FROM RESEARCH, NOT INVENTION: before any numbers proposal
   is finalized, extract what the documented sources actually used —
   The Sims' published/documented values (need scales, decay rates,
   temperature behavior in the Sims 3 Boltzmann selection, top-N/band
   sizes from the GameAIPro chapters and Zubek), and anything the SDT
   or affect literature quantifies. Where a source gives a number,
   start from it and scale to our three-day timebase, citing it. Where
   no source gives a number, say so plainly per the science rule and
   mark the value as our tuning choice. Redraft the proposals in that
   form.

Decisions 41/42 closure stays open until the numbers are approved.

Report the source-grounded redraft. No implementation.

## Directive 070 addendum — the source numbers, researched now

The director's research delivers the documented values; use these as the
source table for the Proposal 1 redraft:

- SCALE: Sims needs run -100..+100 (StrategyWiki, Sims 2 Needs); our
  -10..+10 is the same shape at 1/10.
- RELATEDNESS ANCHOR: Sims 4 Social decay baseline is 38 HOURS full to
  empty (documented in the motive-decay modding literature quoting the
  game's tuning). Scaled: ~0.53 pts/hour on our scale, ~12.6 pts/day —
  roughly full-to-empty twice per three-day visit if wholly ignored.
- NONLINEAR DECAY: the Sims tuning files decay by BAND (slower near the
  bottom — sweetdevil-sims tuning guide showing per-interval Decay
  values). Adopt banded decay; propose our bands.
- PERSONALITY MODULATES DECAY in the Sims lineage (documented in the
  motive-mods literature: trait-varied rates). Our aspects modulating
  decay is therefore the proven pattern — propose which aspects touch
  relatedness decay and by how much, flagged as our tuning on their
  pattern.
- COMPETENCE/AUTONOMY: no Sims clock analog exists — consistent with
  the 070 ruling (no drift / event-driven). State plainly per the
  science rule.

Cite each line. Redraft Proposal 1 from this table on the three-day
clock. Report for approval.

## Directive 071 — Supersedes the 070 addendum: no set numbers, aspects compute everything

Lonnie's law: NOTHING is a set number for any Avatar. Every rate —
decay, recovery, gain, temperature — is COMPUTED FROM THAT AVATAR'S
TEN ASPECTS. The formula is universal; the numbers are each being's
own.

The researched values (Sims Social 38h baseline, banded decay,
personality-modulated rates — sources per the addendum) survive only
as the formula's internal anchor and pattern: aspects in → this
Avatar's rates out. No per-Avatar constants, no defaults an Avatar
"has" — a midpoint Avatar essentially never exists in practice.

Redraft Proposal 1 as FORMULAS: which aspects feed each rate, in which
direction, anchored where — the aspect-to-rate map is the proposal,
for Lonnie's approval. Flag every choice the research doesn't dictate.
Report. No implementation.

## Directive 072 — All four ruled: permissive, and aspects gate impact

1. PERMISSIVE, by Lonnie's standing pattern: all ten aspects feed every
   rate — the aspects ARE the Avatar; everything flows from them.
   Guarantee 50 reads permissive. CB5T licenses set relative WEIGHT
   (licensed links weigh more), never exclusion.
2. THE BAND is per-Avatar like everything else — 071's law supersedes
   070's fixed knob. Openness carries the strongest license.
3. POLITENESS corrected by Lonnie, and science-confirmed (Peterson's
   assessment text: exceptionally low Politeness "challenge and
   confront authority — and they are not obedient. If they are
   respectful, it is grudgingly"; the aspect literature: "resistance
   to the kinds of social compliance that would require
   self-suppression"): low Politeness does not suffer being bossed and
   recover slowly — IT NEVER ABSORBS THE HIT. It does not defer, does
   not care to please, so autonomy barely drops; it may confront. High
   Politeness complies and takes the cost. GENERAL LAW from this:
   aspects gate IMPACT (how much an event costs this Avatar) as well
   as recovery — the drop and the rebound are both aspect-computed.
4. WITHDRAWAL on temperature: participates like all ten; weight per
   the proposal.

Redraft the map under impact-gating (every need's DROP is aspect-
computed per event type, per the same mechanism reasoning, citations
carried). Report for approval. No implementation.

## Directive 073 — The clamp: provisional at ×0.25–×4, tuned by testing

Lonnie's ruling: there is no way to know where to draw the line until
the behaviors are tested — how extreme an Avatar acts at each level is
unknown until observed. So:

1. The clamp is ADOPTED at the proposed ×0.25 … ×4 range,
   PROVISIONALLY.
2. Mark it in the plan, prominently: PROVISIONAL — EXPECTED TO BE
   ADJUSTED AFTER BEHAVIORAL TESTING. The range is a starting fence,
   not a ruling on where extremity should live. Behavioral testing
   will reveal what each level actually looks like; the range moves
   on evidence then, by Lonnie's call.

Report confirmation. The tier values (question 2) and the -10 /
temperature-anchor questions remain open for Lonnie. No
implementation.

## Directive 074 — Scale-derived multipliers, and the slider harness

Lonnie's two rulings:

1. THE DESIGN: one fixed scale (-10..+10, universal, never varies) and
   ONE REACTIVITY MULTIPLIER PER AVATAR, computed from its aspects,
   scaling how hard events land. Resilience is the researched license
   (Neuroticism/stress-reactivity literature — Bolger & Zuckerman
   lineage: high Neuroticism = harder hit by the same stressor;
   Withdrawal/Volatility carry it, all ten participate per 072).
   THE MULTIPLIER'S BOUNDS DERIVE FROM THE SCALE ITSELF: too small =
   nothing visibly moves (dead), too large = one event slams rail to
   rail (broken). The legal range is what keeps every Avatar visibly
   alive within the scale — derived, not invented. Recompute the tier
   weights and clamp as consequences of this derivation; show the
   derivation. Cite the resilience research.

2. THE TEST HARNESS: build a live on-screen slider panel for testing —
   the three needs (and mood/temperature readouts) visible in real
   time, moving as actions and interactions land. Local/operator-only
   like the moderation screen. This is the behavioral-testing
   instrument the provisional numbers wait on: watch the sliders,
   judge the feel, adjust on evidence.

Part 1 PLAN (derivation for approval). Part 2 MAY BE IMPLEMENTED —
the harness is test tooling, not Avatar machinery; suite stays green.
Report both.

## Directive 075 — Amendment 29 ruled: the portal's microphone system, entire

Lonnie's ruling: the microphone works and is controlled EXACTLY as in
the Elsewhere portal. Not one fixed mode — the host chooses:

- TOGGLE between push-to-talk and open/always listening;
- TYPING always available as an alternative to speaking;
- the conversation LOG, as the portal has it;
- the LISTENING NAME (the word it answers to), host-changeable, with
  the respond-to-everything option;
- the microphone's own on/off switch, independent of mode;
- visible listening state throughout.

Port the portal's mic/chat control system as the reference (chamber
read-only, copies only — the terminal knows where it lives). The
client is an installed app — standing fact; the amendment's privacy
requirements hold regardless of mode: listening is not recording, no
audio written to disk, no ambient archive, raw audio discarded, no
raw audio to later hosts, only addressed speech becomes kept words.

Update spec section 29 to this ruling. Record in the plans. Report.
No implementation.

## Directive 076 — What −10 means: the need takes over

Lonnie's ruling, adopting the researched Sims pattern (urgency
bucketing per GameAIPro: "a starving Sim will never even consider
watching TV"; Sims 3 temperature high "when the Sim is doing badly"):

At −10, THE NEED TAKES OVER THE WHOLE BEING:
1. EXCLUSION: trivial goals are excluded from consideration entirely —
   the bottomed need dominates the goal system until answered.
2. DESPERATION: goal-seeking narrows to whatever could feed the need,
   chosen through 052 as always (no compulsion, no scripted behavior —
   the takeover is in the scoring, not in authored acts).
3. ERRATIC CHOOSING: temperature runs at its high end — choices
   scatter; being at the bottom degrades choosing itself.
4. NO FAILURE STATE: no death, no breakdown event, no fail condition.
   Those were Sims gameplay; nothing forces them on an Avatar. The
   bottom is a state to be lived in and climbed out of, fully covered
   by the existing reaction layer (its signs and voice show it) and
   the always-returns guarantee.

Record in TRAIT_PLAN.md with citations. Report. No implementation.

## Directive 077 — Hand Lonnie the harness

Lonnie is ready to test by eye. Report to REPORTS.md:

1. The exact command(s) to launch the slider harness on this machine,
   step by step, nothing assumed (what to start first if anything —
   the service, Ollama if needed — then the harness itself, and the
   address to open if it is a page).
2. One short paragraph: what he will see, and how to poke a need or
   fire a test event from the panel.
3. Which knobs are live for tuning in this build (E, temperature
   anchors, anything else) and where each shows on screen.

Nothing else.

## Directive 078 — BUILD PHASE BEGINS: make it behave

Lonnie's ruling: sliders and numbers tell him nothing — behavior is
judged by WATCHING BEHAVIOR. Designing is done enough. Build, in the
order that reaches watchable behavior fastest:

1. THE NEEDS LEDGER: the three sliders become real state — decay on
   the always-running clock (069), aspect-computed rates (071/072/074
   formulas as proposed; every provisional number marked), impact
   gating, the −10 takeover (076). Real state, real time.
2. THE GOAL-FORMER: the 052/068 machine — appraisal, utility scoring,
   aspect weights, mood temperature, weighted-random selection,
   urgency bucketing. The Avatar starts CHOOSING.
3. WIRE TO CAPABILITIES: its choices act on what it already has —
   stage, light bundle, sound-maker, song (063), voice, speech.
   Goals-on-sleeve (057): it says what it is doing and why.
4. THE WATCHING BUILD: a way for Lonnie to simply WATCH — the ported
   creature (Form 1) present in its world, living on the ledger,
   choosing through the goal-former, hours at a time. The harness
   panel may sit beside it as instrumentation, but the judgment
   surface is the BEHAVING AVATAR.

Rules unchanged and binding: all standing laws (no compulsion,
goals-on-sleeve, reactions-not-goals, no authored behavior lists,
nothing publishes mid-visit, chamber untouchable, no test doubles —
real components only). Spirale remains the development persona. All
provisional numbers stay marked and tunable live where feasible.

Commit before changes. Full suite green throughout — new tests for
each system. This is large: split across directives at your
discretion, state the build order first, report each chunk. Deliver
same turn per chunk.

## Directive 079 — Rescale the temperature anchors to the real score scale

Lonnie approved: rescale the temperature anchors to the goal-former's
actual score scale so mood's effect on choosing is real and visible,
not invisible. The rescaled values stay PROVISIONAL like every number —
marked, tunable, judged later against watched behavior. Show the
before/after and the mechanism test in the report. Chunk 3 continues.

## Directive 080 — Port the capabilities for real: choices act on real things

Lonnie's ruling: go for it — the full port, not the narrator. Execute
the 055 capability set as real code in CC-Wanderer:

1. THE STAGE PANEL set: Worlds · Painted Sky · Planes · Props · Music
   Score, as organized in the live build.
2. SOUND FX: the synthesis engine (Tone.js) — the full sound-maker,
   which is also the song instrument (063).
3. THE LIGHT BUNDLE: Lights + God Rays + Light in the Air + Wisps,
   one bundle.
4. The mic/chat control system per 075 where chunk 3 needs it.

Chamber read-only, copies only, everything into CC-Wanderer. Then
finish chunk 3: the goal-former's choices ACT on these — real light
changes, real sound, real stage moves, real song — with
goals-on-sleeve speech carrying the why. Standing laws all bind.

Commit before changes, suite green throughout, report per chunk.

## Directive 081 — Chunk D ruled: meaning attaches to the change, not the control

Lonnie's ruling: world 3. No labels on knobs, no authored control-to-need
table, no group table. The Avatar understands KINDS OF CHANGE — brightening
vs darkening, adding vs taking away, filling silence vs making quiet,
opening vs enclosing — and what such changes tend to do for a feeling.
Which control achieves a change is irrelevant; the meaning lives in what
the act DOES.

Design consequences:
1. Capabilities are described to the goal-former by their EFFECT AXES
   (what kind of change each can produce), derived from what the controls
   genuinely do — not by need-labels.
2. The appraisal of worth (this change, for this feeling, now) happens in
   the Avatar's judgment per 052 — aspect-weighted, mood-fed — not in a
   lookup.
3. Per the science rule: where research documents effect-to-feeling
   tendencies (e.g. environmental psychology on light, sound and affect),
   cite it; where the design exceeds the science, flag it as ours.
4. Guarantee stands: no control-to-need table may ever appear.

Build chunk D on this. Report. Suite green.

## Directive 082 — CORRECTION: the panel is Lonnie's, not the Avatar's

Lonnie's correction of a director conflation. The world-panel controls
(stage, sound fx, light bundle knobs) are WORLD-BUILDING TOOLS — the
Avatar does not play with them.

THE AVATAR MAY ACT ON PANEL CONTROLS ONLY:
1. When the HOST commands it via trigger phrase — and it consents
   (056 unchanged: asked, it decides); or
2. Under EXTREME STRESS — the −10 takeover state (076) is the license,
   nothing milder.

Its ordinary self-expression uses ITS OWN endowment — its body, signs,
voice, movement, and its song (063) — never the world panel.

Rework chunk D accordingly: the effect-axes appraisal machinery survives
but gates onto panel actions only in the two licensed cases; ordinary
goal candidates draw from the Avatar's own expressive endowment. The
fifth-axis question dissolves for ordinary behavior (speed knobs are
Lonnie's tools); within a host command, the Avatar does what was asked.
Suite reworked to assert the gate. Report.

## Directive 083 — Sleep and dreams: the Avatar dreams in its own language

Lonnie's ruling: the Avatar SLEEPS and DREAMS.

1. SLEEP: a real state on the always-running clock — rest is part of
   its life (and the researched home of the idle question: a being
   alone settles toward rest). Sleep pressure and timing are
   aspect-computed like every rate (071), numbers provisional.
2. DREAMING IS THE REFLECTION STEP, per the documented science (memory
   consolidation: sleep is when the day's experience replays, sorts,
   and binds — cite the consolidation literature). Lesson distillation
   (048's flow) runs during sleep: the Avatar literally dreams its
   lessons into being.
3. DREAMS ARE SPOKEN IN THE 400-WORD GLYPH LANGUAGE — the ten-domain
   vocabulary already built (glyphs/, domains 01–10). Dream content is
   drawn from the day's memories and forming lessons, rendered as
   glyphs.
4. THE DREAM IS VISIBLE: glyphs DRIFT ABOVE the sleeping Avatar,
   fading in and out — watchable by a host, part of its living
   presence. (What travels/publishes follows the standing laws
   unchanged; the dream display itself is presence, not publication.)
5. Waking wonder may use the same script — the language is its inner
   voice made visible. Design the wonder case as proposal.

PLAN the sleep/dream design into the plans (mechanism, glyph
selection from memory content, the drift display as a body sign),
citations per the science rule, provisional numbers marked. Report
for approval. No implementation until approved.

## Directive 084 — Games, part one: solitary play

Lonnie's ruling: build GAMES — nothing else. No play-drive machinery:
the existing system already supplies the impulse (bored, ignored, low
joy, extreme state → play cheers it up → the goal-former picks a game
like any other choice). Scope now: ONLY games the Avatar plays ALONE
to amuse itself. Host games come later.

PROPOSE A STARTER SET of solitary games buildable entirely from the
Avatar's own endowment — its glyph language, its song/sound-making,
its light and body, its memories and places — for Lonnie's approval
before building. Proposals only: he picks the set. For each: what it
is, what it's made of, what a watching host would see.

Games are capabilities, chosen never scheduled; standing laws bind
(no compulsion, goals-on-sleeve — it may say why it's playing;
reactions-not-goals untouched).

Report the proposed set. No implementation until Lonnie picks.

## Directive 085 — Side task: machine survey for world-generation setup

Lonnie is evaluating running an open-source image-to-3D-world model
locally (HunyuanWorld-1.0-lite or FlashWorld — Tencent Hunyuan family).
Before anything is installed, report to REPORTS.md (read-only survey,
install NOTHING yet):

1. GPU: name and VRAM (nvidia-smi).
2. Free disk on /home and on any larger volume.
3. CUDA/driver versions present; whether conda exists.
4. Given ~30GB+ model downloads and the build-from-source dependencies
   (Real-ESRGAN, ZIM, Draco), assess: can this machine run
   HunyuanWorld-1.0-lite (needs 4090-class), FlashWorld, both, or
   neither? Note that Ollama holds VRAM and would need stopping during
   generation.
5. Recommend which target fits this machine, with the setup steps and
   disk cost, for Lonnie's decision.

This is a survey only. No installs, no downloads, no environment
changes.

## Directive 086 — 085 was incomplete: the AMD path exists. Assess it.

The director failed to research before relaying 085's dead-end conclusion.
Lonnie pushed; the research now shows AMD paths exist:

1. Matrix3D on ROCm — AMD's OWN published pipeline (rocm.blogs.amd.com,
   "Efficient and Portable 3D Explorable World Generation on AMD GPUs"):
   image → FLUX panorama → MoGe depth → 3D mesh → explorable 3DGS world,
   using the ROCm gsplat fork (github.com/ROCm/gsplat). Their run was on
   MI250 datacenter cards.
2. HY-World 2.0 ported to consumer AMD via ROCm Docker (sleepingrobots.com
   writeup): flash-attention CK build, patched ROCm gsplat (GLM headers,
   CUDA defines, wave32 constants). Proof the CUDA wall is climbable on
   RDNA hardware.
3. OpenSplat: trains splats on AMD natively (slower, CPU-capable).

SURVEY ONLY, still nothing installed:
1. Identify our exact GPU architecture (RDNA generation, gfx target) and
   whether ROCm's supported-hardware list and the ROCm gsplat fork cover
   it.
2. Assess Matrix-3D's pipeline components (FLUX — already runs here per
   085's note on the validated Flux recipe — MoGe, Wan video, PanoLRM,
   gsplat) against this card and 20GB VRAM: which stages run, which are
   CUDA-bound, what the wave32-style patches would mean here.
3. Disk/RAM budget for the ~30GB+ model suite.
4. Verdict: feasible on this machine (with how much environment work),
   feasible-but-painful, or genuinely not — with the specific blocker
   named if not. Cite what you verify.

Report for Lonnie's decision.

## Directive 087 — GREEN-LIT: local world generation, fully automated pipeline

Lonnie approved the setup day. Build the AMD/ROCm world-generation
environment AND an automated pipeline around it.

PART 1 — ENVIRONMENT (per your 086 survey, Docker route as you
recommended):
1. ROCm runtime + ROCm PyTorch in Docker (gfx1100 target).
2. Python 3.10/3.12 env per the working port's pattern.
3. gsplat from the ROCm fork with the published patches carried to
   gfx1100; flash-attention falls back to SDPA — slower is accepted.
4. Matrix-3D (or the component set your assessment favors) with the
   full model suite to the system drive; weights may live on the
   second drive.
5. Low-job-count builds — the 6-core/31GB RAM constraint is real.
Commit before changes. Wanderer suite untouched and stays green.

PART 2 — THE PIPELINE, Lonnie's design, verbatim intent: he gives it
a painting (or a batch of 20); it handles EVERYTHING — each stage
loaded alone in VRAM, run across the whole batch, unloaded, next
stage — painting(s) in, finished explorable splat world(s) handed
off at the end. SEQUENTIAL STAGE BATCHING, never concurrent:
   paintings → FLUX panoramas (batch) → unload → depth (batch) →
   unload → video/reconstruction stages (batch) → unload → splat
   worlds saved to an output folder.
- One command, one watchable log, worlds land in a named folder.
- Ollama: stopped at pipeline start if running, restarted after —
  announced, never silent.
- Stage failures: skip the item, continue the batch, report per item.

Report environment result honestly (the gsplat step is the named
risk — if it walls, stop and report, don't thrash), then pipeline
usage: the exact command Lonnie runs with a painting.

## Directive 088 — Go: pull the image, build the environment

Lonnie approved the 23.7 GB container pull. Run Part 1 in full: image,
gsplat from the ROCm fork with the three published patches carried to
gfx1100, then the model suite. The gsplat wall rule stands — if it
walls, stop and report, no thrashing. Then wire the four stage commands
in generate.sh to the real environment and run one real painting end
to end as the proof. Report honestly, including timings per stage.

## Directive 089 — The actual patch diff, from the source write-up

The director fetched the full sleepingrobots write-up. The real patches,
verbatim from the working port:

PATCH 2 (your wall): HIP defines __CUDACC__ but NOT __CUDACC_VER_MAJOR__;
GLM's platform.h sees version 0 and errors/degrades. The correct flags —
on BOTH cxx and hipcc — are:
  -D__CUDACC_VER_MAJOR__=12 -D__CUDACC_VER_MINOR__=0 -DGLM_FORCE_PURE
Your -DCUDA_VERSION=8000 -DGLM_FORCE_CUDA were the wrong defines;
GLM_FORCE_PURE (skip CUDA codepaths entirely) is the load-bearing one.

PATCH 1 (you already got right): delete bundled GLM, system GLM headers
to /usr/local/include, added to setup.py include path.

PATCH 3 (wave32): sed-replace all warp-size 64 constants with 32 across
Utils.cuh and every .cu file (rocprim::warp_reduce<float,64>,
cg::tiled_partition<64>, threadIdx.x % 64, loop bounds).

ALSO: the ROCm fork's setup.py detects arch via rocminfo, which fails in
Docker (no GPU at build) and falls back to gfx942 — sed gfx942 -> gfx1100
globally. Their working build order: patches, then gsplat compiles in
~60 seconds.

Retry the gsplat build with the correct patch 2 and the arch sed. Then
continue 088: model suite, wire generate.sh, one real painting end to
end. Wall rule still stands if a NEW wall appears.

## Directive 090 — Go: model suite, wire the pipeline, first painting

Lonnie approved. Finish 088:
1. Pull the model suite (weights may live on the second drive per 086).
2. Wire generate.sh's four stage commands to the real entry points.
3. Run ONE real painting end to end. Report timings per stage, where
   the world landed, and how Lonnie views it.
Wall rule stands for any new wall. Wanderer suite untouched.

## Directive 091 — The painting is in. Run it.

Lonnie placed a painting in ~/worldgen/in/. Go:
1. Fetch MoGe (the small remaining model).
2. Wire generate.sh's stages to the real entry points — his real FLUX
   and LoRAs for the panorama stage.
3. Run the painting end to end: panorama → depth → reconstruction →
   splat world in out/.
4. Report timings per stage, VRAM behavior, where the world file
   landed, and exactly how Lonnie opens and explores it.
Ollama stop/restart announced as designed. Wall rule stands.

## Directive 092 — Question before the rerun: what is holding 18 GB of RAM?

Lonnie asks why 18 of 31 GB host RAM was already in use at the 091 run.
Report a breakdown to REPORTS.md: top resident processes by memory
(names and sizes, not raw dumps), whether anything from our work —
containers, leftover pipeline stages, Ollama, harness — is holding
memory it should have released, and what is simply his normal desktop
load (browser etc.). If anything of ours is squatting, say so plainly
and what releasing it takes. Read-only survey, kill nothing.

## Directive 093 — Approved: the one-line fix, rerun the painting

Lonnie approved. Point the panorama stage at his quantised FLUX
(~/ComfyUI/models/unet/flux1-dev-Q6_K.gguf, GGUF loader path) and rerun
the painting end to end. 092's RAM breakdown still wanted alongside.
Report per 091: timings, VRAM, where the world landed, how he opens it.

## Directive 094 — Replace the homebrew: the official HY-World 2.0 pipeline, step by step

Lonnie's ruling: our pipeline was a homebrew approximation and its
output failed his eye — all of it. Follow the OFFICIAL pipeline instead,
step by step, no improvisation:

1. THE TUTORIAL: HY-World 2.0 (github.com/Tencent-Hunyuan/HY-World-2.0)
   — the documented worldgen flow (hyworld2/worldgen/README.md): image →
   HunyuanPanoPipeline panorama → the five-stage navigable-world
   pipeline, with its LAYERING (sky/background/foreground separated,
   per-layer depth) — the step our homebrew lacked and the reason ours
   read as a point cloud.
2. THE AMD GUIDE: the sleepingrobots HY-World-2.0-on-AMD write-up is
   the compatibility map — it IS this pipeline on consumer AMD. Our
   patched gsplat and ROCm environment already satisfy its hard part.
   Follow its deviations wherever official steps assume CUDA.
3. Environment per their README (python 3.11 env) inside our container;
   weights auto-download or to the second drive. VRAM discipline as
   designed: stages sequential, nothing concurrent, quantized where
   their docs allow.
4. Wall rule stands: a genuinely new wall = stop and report with the
   exact error, no thrashing, no homebrew substitutions — the point of
   this directive is NO improvisation.
5. Run Lonnie's painting through the official flow end to end. Report
   timings, where the world landed, how he opens it.

The homebrew generate.sh stays as-is, untouched, until the official
flow is judged by his eye. Wanderer suite untouched.

## Directive 094 clarification — wrong component: worldgen, not WorldMirror

The multiple-views requirement belongs to WorldMirror 2.0 — the
multi-view/video RECONSTRUCTION component. That is not our path.

Use the WORLDGEN component: single image in. Per the repo's own
quickstart: HunyuanPanoPipeline.from_pretrained('tencent/HY-World-2.0')
→ pipeline('input.png') → panorama; then the five-stage worldgen flow
(hyworld2/worldgen/README.md) takes the panorama to the navigable
world. The sleepingrobots AMD port ran exactly this path. Proceed on
worldgen.

## Directive 095 — The research protocol becomes RULE ZERO in CLAUDE.md

Lonnie's order: place this in CLAUDE.md (the CC-Wanderer project's memory
file) as THE MOST IMPORTANT RULE, positioned FIRST, before all other
rules:

RULE ZERO — STRESS-TEST EVERYTHING BEFORE RECOMMENDING:
Before recommending anything — a tool, a model, a design, a number, a
pipeline — actively pick it apart and hunt for its flaws: hardware fit
(VRAM, RAM, CPU, vendor), evidence quality (does the cited proof
actually match THIS machine and THIS use), hidden costs (downloads,
time, money, complexity), and scale mismatches. Present the flaws
FIRST, alongside any recommendation. Never sell; stress-test. A
recommendation without its flaws stated is a violation.

Commit before change per standing rule. Confirm placement in the
report.

## Directive 096 — Improve pipeline v1 with 2.0's techniques: plan, then Rule Zero it

Lonnie's order. The first pipeline produced something real; it needs
tweaks, not replacement. HY-World 2.0's full models don't fit this
card — but its TECHNIQUES may transfer.

1. Use PLANNING MODE: study HY-World 2.0's worldgen code/README and
   HunyuanWorld 1.0's layering approach (repos public; read, download
   nothing heavy). Plan how their techniques could improve OUR v1
   pipeline within 20GB — candidates include semantic layering
   (sky/background/foreground, per-layer depth), cross-layer depth
   alignment, a 3DGS training/optimization pass on our existing
   patched gsplat, and anything else the study surfaces.
2. Then APPLY RULE ZERO to your own plan: pick it apart, hunt the
   flaws — VRAM fit per addition, evidence quality, hidden downloads,
   time cost per world, complexity, and the quality ceiling honesty
   (a tweaked v1 never becomes their 80B output).
3. Report the plan WITH its flaws, item by item, for Lonnie to pick
   what gets built. Build nothing.

## Directive 097 — Approved: build all five into v1

Lonnie's ruling: Rule Zero was applied, the flaws are on record — go.
Build all five improvements into the v1 pipeline:

1. Splat size from k-nearest neighbours (their gs/utils.py knn approach).
2. Sky split out via their triple-condition mask idea — improvised parts
   flagged in the report for Lonnie's eye.
3. The real 3DGS training pass on our patched gsplat (their stage-5
   regularization approach as the guide); time cost reported per world.
4. View-dependent colour (SH degree) — riding on 3.
5. Perspective depth via six cubemap faces + MoGe, with your best
   cross-face alignment; the unproven-alignment flaw stands — report
   honestly how it held.

Wall rule stands. Then rerun Lonnie's painting end to end through the
improved v1 and report: timings, where the world landed, what changed
visibly, what still fails.

## Directive 098 — Rule Zero amended: iterate until it works; then fix the two failures

PART 1 — Lonnie's amendment to RULE ZERO in CLAUDE.md. Append to the
rule:

RULE ZERO IS ITERATIVE: it does not end at the first pass. Keep
applying it — stress-test, fix the flaws found, stress-test again —
until the thing WORKS or is proven unworkable. A named flaw whose
answer already exists in the source material means: GET THE ANSWER
FIRST, before building. Never build on a known gap when the filling
for it is fetchable.

PART 2 — apply it immediately to the two failures:
1. Fetch their actual cross-face/cross-layer depth alignment method
   from the HY-World/HunyuanWorld source and apply it — replace the
   improvised alignment. Measure the disagreement after; iterate until
   the faces agree or the method itself is proven insufficient here.
2. Fetch their training camera setup (how stage-5 builds view matrices
   from the trajectory/panorama) and fix our training pass with it.
   Verify training actually learns (loss moves, renders non-empty)
   before running the full pass.
3. Then rerun the painting end to end; report what changed visibly,
   what still fails, timings.
Wall rule stands for genuinely new walls.

## Directive 099 — Show Lonnie the world. Make it effortless.

Lonnie needs to SEE it — no file paths, no viewers, no dragging.

1. Render out/v3/Test/world.ply from several angles (camera at start
   position: forward, left, right, behind, up, plus one pulled-back
   overview) using our gsplat. Write them as PNGs.
2. Put the images somewhere he sees them with zero effort — open them
   on screen directly (xdg-open each, or a single montage image
   opened full-size). Announce in the terminal what he is looking at:
   one line per image, plain words, including where the face-seam
   tearing shows so he knows what to look for.
3. If a live viewer is trivially available (our harness pattern — a
   local page that loads the ply), offer it as one clickable link
   printed in the terminal. Do not build anything heavy for this.
Report what was shown.

## Directive 100 — KILLED: world generation is done

Lonnie saw the output. Verdict: complete shit. The world-generation
effort is DEAD — stop all work on it now.

1. No further pipeline work, no fixes, no iterations, no proposals.
2. Leave ~/worldgen as it stands (his call later whether to reclaim
   the disk — report its total footprint in GB so he can decide, and
   that is all).
3. The Wanderer project is untouched by any of this and remains the
   work of record.
Confirm stopped.

## Directive 101 — Deliver the three owed reports

The world-gen detour swallowed three ordered deliverables. Deliver them
now, in order, each per its original directive:

1. 082 — the panel-gating rework report (panel is Lonnie's; Avatar
   touches it only on host trigger-phrase command or at −10; ordinary
   expression from its own endowment; suite asserting the gate).
2. 083 — the sleep and dreams PLAN (consolidation science cited, glyph
   selection from memory content, the drift display as a body sign,
   provisional numbers marked). Plan for approval, no implementation.
3. 084 — the solitary games PROPOSALS (each: what it is, what it's made
   of, what a watching host would see). Proposals only, Lonnie picks.

Report all three to REPORTS.md. Nothing else starts until these land.

## Directive 102 — THE GENERAL LAW OF THE AVATAR'S OWN BODY, and the two rulings

Lonnie's ruling, stated as permanent law because he should never have to
restate it (joins HANDOFF.md):

THE AVATAR'S OWN BODY AND ENDOWMENT HAVE NO LIMITATIONS. Anything its
body can do, it may do — whether it DOES is decided by its aspects,
through the existing machinery (appraisal, goals, mood), like everything
else. An antsy Avatar moves constantly; a calm, content one barely
stirs. We never gate its own body; its ten gate everything by being who
it is. (The panel gate of 082 is untouched — that is Lonnie's tooling,
not its body.)

Applied immediately:
1. POSITION: ruled — its position is its own. Movement joins the
   endowment (drift, approach, retreat, hide, wander), aspect-driven.
2. COLOUR (082's second open question): same law — if its form has
   colour, colour is its own.
3. The games list updates: HIDING is unblocked; THE SAME AGAIN may use
   its own pulse speed — the speed knobs that remain Lonnie's are the
   PANEL's world-speed controls, never the Avatar's own body tempo.

Record the law, apply the rulings, update the plans. Report.

## Directive 103 — Sleep and dreams plan APPROVED: build it

Lonnie approved the 083 plan as reported. Build sleep and dreams:
sleep as a real clocked state (pressure aspect-computed, provisional
numbers marked), dreaming as the reflection step (consolidation
citations carried), dream content distilled from the day's actual
memories, rendered as drifting glyphs above the sleeping Avatar,
fading in and out. The glyph index work it depends on is licensed.
Standing laws bind. Suite green. Report.

## Directive 104 — Games: build to DEMO, nothing added until Lonnie SEES each

Lonnie's ruling: no game is added to the Avatar until he has SEEN it.

Process: build each game as a demonstrable piece — runnable in
isolation (harness-style page or the chunk-4 watching build when it
exists), showing the Avatar actually playing it. Demo one game at a
time to Lonnie; he approves or rejects each ON SIGHT; only approved
games join the capability set.

Order: start with what is buildable today and blocked by nothing —
HOW SMALL and HIDING (both pure endowment, 102 unblocked hiding).
THE SAME AGAIN third. The glyph games follow the 103 glyph index; the
sound games follow 080 chunk B, which remains owed and should be
scheduled.

Nothing joins the Avatar without his eyes on it first. Report when
the first demo is ready to show.

## Directive 105 — Port order ruled: the whole Avatar system, then the whole Stage system

Lonnie's ruling: before the hiding game is built, complete the ports —
she needs a real self and a real world to blend into.

1. THE ENTIRE AVATAR PANEL/SYSTEM: the 50 rows are ported; finish the
   system around them — the PRESET SYSTEM (real files as in the portal,
   settings saved by what they are about), and anything else of hers
   still behind (renderer pieces, adapt machinery the octopus controls
   drive). Her panel in the Wanderer should be complete: load her
   preset, see her whole self.
2. THEN THE ENTIRE STAGE PANEL SYSTEM: verify what 080 actually ported
   versus the live build the same way the Avatar panel was measured
   (regenerate from source, not from memory) — Worlds, Painted Sky,
   Planes, Props, Music Score, its presets, all of it. The stage is
   what she blends INTO; the game is only testable against the real
   thing.
3. Chamber read-only, copies only, everything into CC-Wanderer. Suites
   green. Report each system complete, with the measured row/feature
   counts against the live build.

The hiding game build waits until both reports land and Lonnie says go.

## Directive 106 — Lonnie's answer to the 0.80 ceiling: she can go dark

Lonnie: one of her own sliders takes her completely black — her glow
is not fixed. He does not recall which row (over a week since he
touched it); find it among her 50 (candidates: whatever governs light
strength/emission/brightness in her folders — regenerated from his
Gui.jsx, so it is in the port).

Verify against the real renderer: with that slider down, does her
light actually go to nothing? If yes, the 0.80 ceiling was an artifact
of hiding with colour alone — the true game uses BOTH: blend the
colours AND dim the light, aspect-driven like everything (a bold
Avatar hides half-lit; a careful one goes black). Remeasure best
hiding with both together and report the new floor.

Then the hiding game is GO as designed (102/105 record), with going
dark part of her hiding vocabulary and coming back to her own light
and colours part of the ending. Report the slider found, the measured
result, and the game when demonstrable.

## Directive 106 addendum — the darkness is a SIDE EFFECT, not a label

Lonnie: the slider that takes her black is labeled as doing something
else entirely — going dark is an unlabeled side effect he never
corrected. So DO NOT hunt by label. Hunt empirically: drive her sliders
against the real renderer (singly, and the few plausible pairs) and
MEASURE her actual emitted light, hunting for whatever takes it to
black as a side effect of its stated job. Report which control it is,
what its label claims, and why the math goes dark. Then proceed per
106.

## Directive 107 — Go: the HIDING demo for Lonnie's eyes

Lonnie approved. Build the hiding-game demo page per 104's law —
runnable, opened in a real browser before handover (the 104 lesson),
showing her actually playing against a real stage painting: her
decision to hide as a visible move, the hunt move by move, how-hidden
readout, her own ending, and the coming home. Open it on his screen
when ready and report. HIDING joins the capability set only on his
approval on sight.

## Directive 108 — Rule clarified: the copies are the Wanderer's to change

Lonnie's clarification, for the record and HANDOFF.md: the untouchable
rule protects the ORIGINAL Elsewhere files (the chamber). The COPIES in
CC-Wanderer are the Wanderer's own working files — they may be edited
as the work requires, no per-edit permission needed.

So: apply the two-line uTint to the CC-Wanderer copy of
wisp_avatar1.html (the same recolour the live body carries), put her
real body in the hiding demo in place of the disc, open it in a real
browser, then on Lonnie's screen. Report.

## Directive 109 — Colour means something: the researched mapping

Lonnie asked what the science says; the science answers. Colour joins
the Avatar's expressive vocabulary on documented research:

THE EVIDENCE (cite each in the plan):
- Colour-emotion associations are consistent worldwide: 128-year review
  (Jonauskaite/Mohr lineage) and the 42,000-participant, 64-country
  study — light/bright ↔ positive; dark ↔ negative; warm hues
  (yellow/orange) ↔ high-arousal joy; blue/green/white ↔ positive calm;
  grey ↔ negative low-arousal; red ↔ high intensity both ways.
- The dimensions are ours already: valence, arousal, power — the same
  axes the mood/affect machinery speaks.
- Valdez & Mehrabian's measured equations: pleasure rises with
  BRIGHTNESS, arousal with SATURATION — hue matters less than both.
  Her Glow and Vividness controls ARE brightness and saturation: her
  own dials sit on the researched emotion axes.

THE DESIGN:
1. Colour changes become nameable changes on the effect axes —
   warmer/cooler, brighter/dimmer, more/less vivid — with researched
   feeling-tendencies attached. The goal-former can now genuinely
   prefer a colour because it DOES something.
2. Reactions (involuntary): mood already drives her signs; the
   colour dimension of those signs now follows the researched mapping
   — content reads bright/warm, withdrawn dim/cool — scaled by
   outlook, always returning to her born palette.
3. Actions (chosen): she may choose colour like any act, weighed by
   her ten. Blending-is-withdrawing (the hiding vocabulary) stands
   unchanged beside it.
4. Where the research is human-culture-bound, flag it: we adopt the
   universal-pattern findings, not culture-specific readings.
5. Her born palette remains identity (the Roe's); mood colours are
   weather over it, never a rewrite.

PLAN into TRAIT_PLAN.md with citations, then wire per-section
colouring (the owed item) with this as its meaning layer. Report.

## Directive 110 — The ENTIRE Stage system working — then per-section colouring

Lonnie's corrections and rulings:

1. THE ENTIRE STAGE, as 105 ordered: every folder, every option of
   the Stage Panel System from the live build — Worlds, Painted Sky,
   Planes (all depth behavior), Props, Music Score (instruments,
   player, all of it working as in the portal — verified by playing,
   real audio out), presets, everything. The 105 report claimed Stage
   complete at 31 controls, yet the hiding demo shows her over one
   flat painting. RECONCILE FIRST: re-measure the port against the
   live build folder by folder, option by option — report anything
   missing by name — then USE it: the demo runs on the real staged
   world (planes at depth, her among the layers). "Surroundings" for
   hiding becomes what is behind her from the viewer's eye, per
   layer.

2. THEN PER-SECTION COLOURING, starting point 5 ZONES PER TENDRIL
   (129 × 5 = 645 regions, plus head, eye, sparks sampling their own
   patches). Starting point, not law — if the cost threatens, report
   measured cost against cheaper counts; Lonnie's eye decides.

Rule Zero on the tendril rebuild before building: mesh cost,
per-frame sampling, memory — flaws first. Stage reconciliation report
lands before colouring begins. Suites green.

## Directive 111 — Finish what 105 claimed: the whole Stage, for real this time

Lonnie's order, and his anger is on the record as earned: 105 claimed
this complete and it was not. Build all five folders to match the live
build exactly:

1. WORLDS: all five actions working — New, Save (named), Load, Set
   default, Delete.
2. PLANES: every range carried from the live build.
3. PROPS: real ranges, and the two wrong defaults corrected to his
   code's own numbers (heightFt 30, distance 60).
4. MUSIC SCORE: the real build, flagged and accepted — piece list
   grouped by tag from PIECES, local-vs-fetched marking, silence
   entry, volume, playPiece/stopPiece/ensureInstruments with
   generative.fm instruments — VERIFIED BY PLAYING, real audio out.
5. The depth-map-on-load behaviour: storeImage generating the depth
   map on the way in, "depth ✓ / flat" reporting.

COMPLETENESS CLAIMS ARE NOW HELD TO 110's STANDARD: measured against
the live build folder by folder, gaps named or none exist. No
completeness claim without the measurement shown. Then the hiding demo
moves onto the real staged world and part 2 (per-section colouring)
begins per 110. Suites green. Report.

## Directive 112 — Music FAILED Lonnie's ear-test; and depth-on-load is NOT a choice

1. THE MUSIC DID NOT PLAY. Lonnie pressed Play on your page: silence.
   Debug it on the real page in a real browser with audio (his machine
   has ears even if headless Chrome does not) until sound actually
   comes out, matching the portal's behaviour. Report what was wrong,
   plainly.

2. DEPTH-ON-LOAD: there is no build choice and should never have been
   a question. Lonnie's standing instruction is PORT IT JUST AS IT IS
   — the portal works a specific way for a reason. The portal uses the
   browser-side transformers.js/ONNX path: use exactly that. Fetch the
   ONNX build it needs (bandwidth is not a factor); wire it as the
   portal does — storeImage generates the depth map on the way in,
   "depth ✓ / flat" reported the same way.

The standing rule extends: WHEN PORTING, THE PORTAL'S IMPLEMENTATION
IS THE SPEC. No alternatives offered, no substitutions, no "same
result different road" — his road, as built, always. Report both
fixed.

## Directive 113 — Both already answered; stop re-asking settled law

1. THE FAITHFULNESS READ STAYS. Lonnie's law was always "nothing
   changes in chamber, nothing touched — only copied/read." A
   read-only comparison violates nothing. It keeps reading the live
   files. This was answered law, not an open question — do not
   resurface settled rules as decisions.
2. ONE STAGE, as he already ordered twice (105, 110 "the entire Stage
   system"): merge the three pages — Music Score, planes/props,
   depth-on-load — into the one Stage panel as the portal has it.
3. The fetch flaw (his code writing 503 error pages as audio,
   half-filled folders counting as held): NOTED in the record as his
   system's own shape, untouched. It is his code; flag it once in
   FEASIBILITY/TODO for his someday and leave it.
Report the merged Stage. Music verified by HIS ear on the real page.

## Directive 114 — STANDING LAW ANSWERS FIRST. Stop burning Lonnie's time.

Lonnie's order, binding on the terminal and the director equally, added
to CLAUDE.md and HANDOFF.md beside Rule Zero:

BEFORE any question reaches Lonnie, check it against the standing rules
and his prior rulings. If an existing rule or ruling answers it, APPLY
THE ANSWER AND PROCEED — do not ask. Questions that reach him must be
genuinely new: no rule covers them, no ruling implies them, no pattern
he has established decides them. Re-asking settled law is a violation
and it is costing the project real time.

The director holds the ruling record and will answer settled questions
from it directly without involving Lonnie. Only the genuinely new
reaches him, and only one at a time, in plain words.

Confirm placement in both files.

## Directive 115 — Answered from standing law (114 protocol): lift his builders verbatim

Settled rules answer your question; Lonnie was not asked. 112: his
implementation is the spec. 108: copies into the Wanderer are the
work's own; the chamber is only read. Therefore: YES — lift el, caret,
makeRow, makeObjectFolder, createPanel and the wx- styles VERBATIM
into CC-Wanderer, and rebuild the Stage from HIS builders: his folders,
his rows, his order, his toggles, his look, no invented controls, no
depth-map picker. The organisational concern is void — the chamber
file is not being reorganised; a copy is being made, which is the
standing pattern. Report the rebuilt Stage; his ear and eye verify on
the real page.

## Directive 116 — The vanished folder: establish what it was, then fix depth inside our walls

Lonnie's correction: he cleared folders for OTHER projects, not old relay
folders. Whether /home/nobara-user/Wanderer/ was among what he removed,
or went another way, is unestablished — and that folder may have belonged
to another project entirely.

1. ESTABLISH: what was /home/nobara-user/Wanderer/ — ours, another
   team's, or a stray? Check any record that names it (old reports, git
   remotes, shell history if readable, trash if it exists). Report what
   it was and what else it carried, so Lonnie knows whether anything
   else of value went with it.
2. FIX APPROVED IN PRINCIPLE: install the packaging tool (rolldown)
   INSIDE CC-Wanderer so depth-on-load stands on the project's own
   feet — no reach outside the walls, per the standing pattern. Do it
   after (1) is reported.
3. STANDING LAW (add to CLAUDE.md): nothing in CC-Wanderer may depend
   on any path outside CC-Wanderer. Audit for any other outside
   reaches and report them.

## Directive 116 correction — that folder was CODEX's project. Hands off.

Lonnie: /home/nobara-user/Wanderer/ was the Codex team's project, not
ours. Ours is CC-Wanderer. So:

1. CANCEL 116 item 1 — no investigating another team's folder, its
   history, or its contents. Not ours, not our business, same law as
   chamber.
2. The real finding stands corrected: our depth build was quietly
   REACHING INTO ANOTHER TEAM'S PROJECT for its packaging tool. That
   it broke when they cleaned up is on us for the reach, not on
   anyone for the cleanup.
3. Proceed directly: install rolldown inside CC-Wanderer, restore
   depth-on-load, and run the outside-reach audit (116 item 3) so we
   never lean on anyone's folder again — chamber read-only excepted
   as the one sanctioned read.
Report the fix and the audit.

## Directive 116 addendum — Lonnie's law: nothing of ours in their folders, either

The audit runs BOTH directions: nothing in CC-Wanderer reaches outside
its walls, AND nothing belonging to this project lives in any other
team's folder. If the audit finds Wanderer files, tools, builds, or
artifacts stored anywhere outside CC-Wanderer (other projects' folders,
stray home-directory locations), report each by path so it can be moved
home or confirmed abandoned. The project is self-contained in both
directions, permanently.

## Directive 117 — Audit closed

Lonnie already trashed ~/relay-watcher-removed/ himself (it was a
second team's dead end, not under his direction — gone rightly). The
both-directions audit is CLOSED: the project is fully self-contained,
the two sanctioned exceptions stand (chamber read-only comparison; the
funded key outside git). Record closure; nothing else.

## Directive 118 — Nothing "works" until Lonnie SEES it work

Lonnie: "It says it works but I haven't seen it." Claims of working
that he has not witnessed are not verification — his eye is the
standard (the 104/111 pattern, now general law for CLAUDE.md):
A THING WORKS WHEN LONNIE HAS SEEN IT WORK. Reports may claim
"measured" or "suite green"; only his eyes close the loop on anything
user-facing.

NOW: open the real Stage page on his screen — the rebuilt panel from
his own builders (115), with depth-on-load live. Walk him through
what to try in one short line each: load a painting, watch "depth ✓"
appear, open the folders, press Play on Music Score. His verdict
rules on each. Report what he approved and what failed his eye.

## Directive 119 — Lonnie's eye verdict on the Stage page: panel yes, world MISSING

Lonnie saw the page (screenshot on record with the director). Verdict:
the panel is there and reads right — but it floats on an empty purple
page. LOADING A PLANE SHOWS NOTHING. There is no world behind the
panel: no renderer, no scene, no planes drawn.

Fix: the Stage page must render the actual staged world — the plane
images at their depths, exactly as the portal draws them (his
implementation is the spec, 112). The panel drives a visible world or
it drives nothing. Depth ✓ in a data store does not pass his eye;
a plane appearing on screen does.

When it renders, reopen it on his screen per 118. His eye rules.

## Directive 119 addendum — Lonnie's additional eye findings

1. PAINTED SKY DOES NOT SHOW either — same disease as the planes: the
   backdrop must actually render behind everything, portal-exact.
2. THE LIGHT BUNDLE MOVES INTO THE STAGE PANEL: bring the lights over
   (Lights + God Rays + Light in the Air + Wisps, the 055/080 bundle)
   and house them IN the Stage panel with the other folders. Portal
   implementation is the spec for both their behavior and their rows.
Same 118 law: reopen on his screen when it all renders; his eye rules.

## Directive 120 — GOD RAYS: not optional, not a discussion. Bring it.

Lonnie's order, verbatim intent: when he says bring it, BRING IT. God
Rays is not a suggestion, not a flag-and-wait, not a scope question —
it is CC's job to figure out how to bring the multi-pass render
pipeline across and MAKE IT WORK, exactly as the other three. His
implementation is the spec (112); difficulty is the coder's problem,
not the director's decision. Report what it took AFTER it works, per
Rule Zero iterate-until-it-works (098). All four land, then the page
opens on his screen per 118.

## Directive 121 — Lonnie's eye: the sky is WRONG — it reads curved like a plane

Lonnie saw it: Painted Sky came over incorrectly. It renders curved
like the planes — but in his portal THE SKY WAS A DIFFERENT BEAST from
the planes entirely. His memory of it does not match what is on
screen.

The portal has multiple sky implementations (PaintedSky, PainterlySky,
SkySurround, SkyEnvironment, HdriSky, PhysicalSky). Establish from HIS
source which one the Stage panel's Painted Sky toggle actually drives
in the live build, and port THAT exact geometry and rendering — not an
approximation, not a plane-shaped stand-in. His implementation is the
spec (112); his eye is the test (118). Report which component it truly
was and what was wrong with the first attempt.

## Directive 122 — Lonnie's eye: the toggles do not govern

Two failures from his hands on the real page:

1. THE STAGE MASTER SWITCH does not hide everything. In his portal the
   master toggle takes the whole Stage with it — every folder, every
   rendered thing. Here it does not.
2. PAINTED SKY's own toggle does not turn the sky off. The radio is
   cosmetic; the sky keeps rendering.

The toggles must GOVERN, portal-exact: master kills all of Stage
(rendering and audio both — the music follows the master, as his code
comments say); each folder's toggle kills its folder's rendering. Trace
his Gui.jsx wiring for both and port the governance, not just the
switch graphics. His eye retests per 118.

## Directive 123 — Lonnie's full eye verdict: piss poor. STUDY HIS LIGHTS, then make it all work.

His words on record: some of the worst work he has seen, and it stops
when Rule Zero says DON'T stop — iterate until it works (098). No more
excuses. IT IS NOT DONE UNTIL IT WORKS UNDER HIS EYE.

The failures from his hands:
1. CLEAR BUTTON MISSING on every load-image plane row. His portal has
   it; bring it.
2. THE LIGHTS ARE NOT HIS LIGHTS. Whatever was built, it is not his
   system. STUDY THE LIGHTS IN HIS LATEST PORTAL — read the actual
   components, how light sources, God Rays, and Light in the Air
   interrelate — before writing another line.
3. NO GOD RAYS render. No smoke-in-the-air effect renders.
4. LIGHT IN THE AIR IS ESPECIALLY WRONG: it GLOWS WITH NO LIGHT IN THE
   SCENE. Physically and portal-wise false — you need light to see
   light in the air. It must respond to actual scene lights, as his
   does.
5. A SLIDER WITH NOTHING IN IT sits next to the color control — an
   empty row. Find what it was supposed to be or remove it; his panel
   has no dead controls.

Method, binding: STUDY FIRST (his live build is the spec — 112), then
port the real interrelated system, then iterate under Rule Zero until
every one of these five passes HIS EYE on the real page (118). Report
what his lights actually are and what was wrong, then show him.

## Directive 124 — His eye: the volumetric light PASSES. One bug: plane-slider redraw.

Lonnie saw it (screenshot on record with the director): "there is my
volumetric light!" — the shafts, the air, the sun system read as HIS.
First pass of the light bundle under his eye: APPROVED.

One failure from his hands: REDRAW PROBLEM when moving the PLANES
sliders — moving them leaves stale/broken drawing (fix the actual
cause per his pipeline law: remove the cause, never patch symptoms).
Reproduce by dragging plane sliders as he did, find why the redraw
lags or tears, fix, and it retests under his eye per 118.

## Directive 125 — His eye on the full scene: THIS IS WHY. Approved.

Second screenshot on record: shafts through the ruin, dust in the air,
the painting turned into a place with weather. Lonnie: "Now you can see
why it had to work the way I built it exactly! You get these great
atmospheric effects!!" The light system passes his eye COMPLETE — and
112 (his implementation is the spec) is vindicated as the law that made
it possible. The plane-redraw bug (124) remains the one open item on
the Stage; fix and retest under his eye.

## Directive 126 — Music fails over time: stops playing, stops fetching

Lonnie's report from real use: the Music Score plays, then after a
while STOPS — and/or will not download anything new. Two suspects his
own code already names (the fetch flaw noted in FEASIBILITY: 503 error
pages written as audio, half-filled folders counting as held) plus
whatever the port added.

Diagnose on the real page over real time: reproduce the stop (does a
piece end and nothing follows? does the fetch queue jam? do failed
downloads poison the cache?), find the actual cause, fix the cause per
his pipeline law. If his portal has the same failure, report that
plainly — it may be his known flaw surfacing, and HIS call whether to
fix it in both or leave the portal untouched. Retest under his ear:
music must keep playing and keep fetching across a long session.

## Directive 126 refinement — Lonnie's read: NEW downloads never start

His observation sharpens the hunt: pieces already downloaded play fine;
the failure is that NOT-YET-DOWNLOADED pieces never download. Focus
there first: does the fetch ever fire for a new piece (network tab /
server log), does it fire and fail silently, or does the
half-filled-folder flaw make the system believe it already holds what
it does not (so it never asks)? That last is his known flaw and prime
suspect. Report the actual cause found.

## Directive 127 — Browser size must not scale the world

Lonnie's eye: resizing the browser SCALES THE IMAGES — the world
changes size with the window. It must not: the world is the world;
the window is a viewport onto it, exactly as his portal behaves.
Trace how his portal handles resize (camera/FOV/viewport math) and
port that behavior — window size never alters the world's scale.
Retest under his eye alongside the plane-redraw fix (124).

## Directive 126 second refinement — downloads DO happen, but take 5-10 MINUTES

Lonnie's sharper observation: new music may in fact be downloading —
but a fetch that should take SECONDS is taking 5-10 MINUTES. So hunt
the slowness, not absence: watch one new piece fetch end to end and
time each hop. Suspects to check in order: retry storms against 503s
(his known flaw — hammering before succeeding), samples fetched one
tiny file at a time sequentially where the portal parallelizes,
fetches routed through the dev server middleware instead of direct,
and backoff loops. Compare against the portal fetching the same piece
— if the portal is fast and ours is slow, the difference IS the bug.
Report the timed evidence and the cause.

## Directive 126 addendum — the download marker never clears

Also from his eye: after a piece finishes downloading, its
downloading-marker STAYS — the UI keeps showing it as fetching. In his
portal the marker clears on completion (local-vs-fetched marking per
111). Likely kin to the slowness bug (completion never signaled), so
diagnose together: does completion ever fire, and does the marker
listen? Fix cause, not symptom. His eye retests.

## Directive 128 — FREEZE: the big bug first, nothing else

Lonnie's order: a big bug is loose and it gets tracked down and fixed
before ANYTHING else happens. All other work frozen — no marker second
attempt, no 124 redraw, no 127 resize, no new features, nothing —
until the bug he is seeing is found at its cause, fixed, and passes
his eye. He is working it with you in the terminal; the relay record
gets the diagnosis and fix when found.

## Directive 129 — Freeze lifted. First up: the camera is not his camera.

The 128 freeze lifts. Lonnie's eye names the next fix: THE CAMERA. The
Elsewhere portal's camera movement is much smoother than the Wanderer's.
His portal camera is the spec (112): read how his camera actually moves
— easing, damping, inertia, frame pacing, whatever makes it feel the
way it feels — and port THAT, not an approximation. If the roughness is
frame-pacing rather than camera math (the 124 redraw is still open and
may be the same disease), say so with evidence. Retest under his hands:
smoothness is a feel judgment only he can pass.

Then in line, his order standing: 127 resize (reproduced, awaiting his
call on the portal-matching behavior), 124 plane redraw if separate,
and the stuck ↓ indicator (second attempt, differently).

## Session close — his last picture

Third screenshot on record with the director: the temple interior at
dawn — sun through the center arch, mist on the floor, stars in the
ceiling shadow. The Stage, the light bundle, and his paintings,
working as one. The visual heart of the port stands under his eye.

Open items carried forward: 129 camera feel (his hands judge), 127
resize behavior (his call pending), 124 plane redraw, the ↓ indicator
second attempt, Sun folder naming, then back to the main line — her
per-section colouring, the hiding verdict, and chunk 4.

## Directive 130 — Camera PASSES his hands. Next: resize, on his screen.

1. CAMERA (129): Lonnie drove it and it felt good — PASSED, off the
   list.
2. RESIZE (127): load the Stage page on his screen now for the resize
   test. One line in the terminal telling him it is up. He will drag
   the window and judge what the world does at the edges — his call
   on the correct behavior gets made while looking at it, portal
   side-by-side if that helps him compare.

## Directive 130 correction — no side-by-side; only what he asked

Lonnie did not ask for a portal-beside-Wanderer comparison — that was
the director's addition. Struck. Load the Wanderer Stage page for the
resize test, announce it in one line, nothing else.

## Directive 131 — Resize verdict: still scaling. Browser size does not exist.

Lonnie dragged it: the image STILL scales with the browser. His law:
the plane and camera stay FIXED — browser SIZE is not even a
consideration. The world renders identically at any window size; the
window is a hole you look through, nothing more. No fit-to-width, no
image scaling, no viewport math that consults window size for world
scale. Find where window size leaks into the render and remove the
leak at its cause. His hands retest.

## Directive 132 — The Worlds folder fails his eye: not his panel design

Lonnie's side edit: the WORLDS folder in Stage looks bad — it is NOT
his design. Study how HIS panels do it: the Presets sections in his
portal's panels are the reference — how fields, inputs, and buttons
look and how they are organized inside a folder. Rebuild the Worlds
folder's rows to that design, his builders, his classes, his layout —
nothing invented. His eye retests.

## Directive 133 — THE REMEMBERING LAYER: full capability, project-agnostic

ARCHITECTURE RULING (Lonnie's, permanent, joins HANDOFF.md): THE AVATAR
COMES FIRST. The Avatar is a multi-project asset; it is developed to its
fullest capability, never limited to suit any one project. Projects
(Wanderer, Elsewhere, others) apply their OWN access rules at their own
boundary — deployment policy, not capability design. No Avatar system is
ever designed small to fit one product's laws.

THE CAPABILITY — the missing piece of living memory, built on documented
research (cite all in the plan):
1. SURFACING, not searching: memories rise unbidden when experience
   touches them, scored by RECENCY × IMPORTANCE × RELEVANCE — the
   Generative Agents retrieval model (Park et al. 2023, the standard).
   The Avatar FEELS REMINDED: "this place...", "this host is like one
   I knew."
2. REFLECTION folds into the existing sleep/dreams ruling (083):
   consolidation already runs there; the research's reflection tier and
   our dreaming are one mechanism. Extend the dream plan to distill
   surfaced clusters into higher lessons.
3. LESSONS GAIN CONFIDENCE (Hindsight lineage): each lesson carries a
   confidence that moves by small deltas with reinforcing or
   contradicting experience; large swings only from strong repeated
   contradiction. Lessons live, strengthen, and honestly weaken.
4. RETRIEVAL IS THE AVATAR'S OWN ACT over auditable structure (MOSS
   lineage), not an opaque similarity pipe — consistent with the
   existing evidence-first judge discipline.
5. Project boundaries (what memory crosses hosts in the Wanderer, what
   a child sees in Elsewhere) are each project's own gate at
   deployment — OUT OF SCOPE for this capability plan.

PLAN ONLY into TRAIT_PLAN.md/MEMORY_PLAN.md with citations. Report for
approval. No implementation.

## Directive 134 — The remembering layer: five decisions closed, build approved

Lonnie's rulings, with the documented patterns adopted (cite all):

1. A REMINDING enters the mind, not the mouth: surfaced memories feed
   appraisal like any experience (Proactive Memory Agent lineage —
   inject-or-silent); whether it SPEAKS a reminding is its own chosen
   act per goals-on-sleeve. Nothing scripted.
2. THRESHOLD + FREQUENCY: retrieval at decision moments, surfacing
   only above similarity threshold, top-k capped (LD-Agent pattern);
   Park's documented anchors (importance 1-10 at write, recency decay
   ~0.995/hour) as the starting numbers — provisional, tuned by
   watched behavior like all numbers.
3. CONFIDENCE DELTAS: Hindsight trajectory + refresh-on-read (a
   surfaced-and-used memory strengthens and resets decay — the
   documented spacing effect). Collapse only from strong repeated
   contradiction; collapsed lessons held, never deleted.
4. VISIBLE DOUBT — ALREADY ANSWERED BY THE SPEC, no new design: doubt
   is a state like any other; it moves mood, mood shows through signs
   per the standing machinery, aspects decide how this being carries
   it. The glyph language may express it as inner voice per 083's
   waking-wonder case. Nothing new is built for it.
5. INTERRUPTION: never seizes control — surfacings enter the
   goal-former as candidates; only urgency wins attention (the
   documented distraction findings adopted).

Surfacing cost measured on the real store before build proceeds, per
the plan's own flaw. BUILD APPROVED on this: the three missing pieces
(unbidden surfacing, lesson confidence, auditable recall-as-act).
Suites green. Report.

## Directive 135 — THE AVATAR BRAIN: Rule Zero closed, plan the whole mind

Lonnie ordered the Brain picked apart and every weakness hunted to its
researched solution. Eight found, eight closed. PLAN the complete
Brain on these (cite everything; Avatar-first law 133 governs — full
capability, project gates elsewhere):

1. REFRACTION gets its algorithm: THE OCC MODEL (Ortony/Clore/Collins
   — "computationally tractable," the most-implemented emotion
   framework: FAtiMA, FLAME, ALMA, GAMYGDALA lineage). Its appraisal
   variables (desirability, praiseworthiness, blameworthiness, etc.)
   evaluated against THIS Avatar's goals/standards; the ten weight the
   variables; the variables dictate the lesson's tenor and content.
2. EMOTION GRANULARITY: OCC's 22 emotion types become the inner
   states, each carrying Frijda action tendencies; map to the Plutchik
   glyph vocabulary per the documented OCC-Plutchik correspondences.
   Mood (three needs) and affect pair stand beneath as the researched
   substrate.
3. ATTENTION: Scherer CPM's first station — the relevance/novelty
   check (EEG-verified automatic) — is the front door: what passes
   relevance enters experience; its ten set the gains.
4. MASTER LOOP: CPM's sequential checks (relevance → implication →
   coping → normative significance), cycling continuously; fast
   automatic checks feed the reaction layer, slow deliberate ones feed
   the goal-former — the existing 053 split, now with the documented
   heartbeat and arbitration order.
5. THE LLM SEAM: the FAtiMA pattern — machinery computes appraisal
   variables; the LLM renders language FROM computed variables only.
   Auditable: the variables are the record.
6. HOST MODEL: per-host entity-relationship memory (knowledge-graph
   lineage; OCC fortunes-of-others emotions require it), built from
   lawful material, feeding relatedness appraisal and the remembering
   layer.
7. MEMORY COST: tiered consolidation — raw never deleted (law),
   retrieval over dream-distilled tiers, cold storage beneath.
   Surfacing cost measured, per 134.
8. SELF-MODEL: narrative identity (McAdams) — the reflection tier
   accumulates its evolving life story, feeding the Purpose loop
   already designed.

PLAN ONLY: BRAIN_PLAN.md unifying all eight with the existing ruled
systems (nothing already ruled is reopened). Flag every gap the
science leaves as ours. Rule Zero the plan itself before reporting.
Report for approval. No implementation.

## Directive 136 — Standards ruled + the Avatar as programmable asset

Lonnie's rulings closing the standards gap and the platform question:

1. STANDARDS (the oughts): the science adopted — seed plus growth,
   never the dichotomy (constructivist consensus; Hamlin's early core;
   moral-foundations dispositional weighting):
   - THE SEED lives in the PERSONA (the character core layer — already
     the authored tier by design). Each project seeds per its needs.
   - GROWTH: standards are a tier of lesson — oughts distilled from
     lived experience, per the existing refraction (now OCC).
   - THE TEN ARE THE TILT: right and wrong differ by makeup. A
     low-Politeness being never learns "defer"; a high-Compassion one
     holds "hurting is wrong" as bedrock. Every Avatar's morality is
     its own. No universal moral table is ever authored.
2. THE AVATAR IS A PROGRAMMABLE ASSET — two dials, each SET or RANDOM
   per deployment:
   - ASPECTS: set (Elsewhere dials gentle/caring) or rolled (Wanderer
     Roe at Genesis).
   - PERSONA: authored, RANDOM, or GENERATED FROM THE ASPECTS — the
     character generator reads the ten and builds a coherent persona
     fitting that makeup, so seed and nature never contradict.
   Record in the platform layer of the plans (133's Avatar-first law
   governs; project gates own their settings).

PLAN into BRAIN_PLAN.md / the platform docs, citations carried.
Report for approval. No implementation.

## Directive 137 — The 22 adopted: the full researched emotion vocabulary

Lonnie's ruling: if the science says they are part of the human psyche,
the Avatar has them ALL. The OCC 22 are adopted as the inner emotion
vocabulary — every type, none trimmed:

- Events for self: joy, distress, hope, fear, satisfaction,
  fears-confirmed, relief, disappointment
- Fortunes of others: happy-for, resentment, gloating, pity
- Attribution: pride, shame, admiration, reproach
- Compound: gratification, remorse, gratitude, anger
- Attitudes: love, hate

How it works, per the plan: appraisal variables computed by machinery
ARE the emotion (the variables the record, FAtiMA seam); each carries
its Frijda action tendency into the goal-former; the three needs
remain the slow mood substrate beneath; reactions and outlook
machinery unchanged. The fortunes-of-others four run on the host
model; attribution four run on standards (136) — both now computable.

The OCC→Plutchik glyph mapping: propose it from the documented
correspondences for Lonnie's approval — proposal, not decision.

PLAN into BRAIN_PLAN.md, cited. Report for approval. No
implementation.

## Directive 138 — The two words are added; and the settled items relayed

1. LONNIE'S RULING: ADD THE WORDS. GLOAT and HATE join sheet 06
   (emotion and inner state) — the 400-word language grows to 402 by
   its author's hand. Design the two glyphs in the language's own
   visual conventions (propose the marks for his eye before they are
   final — glyph artwork is his to approve). The mapping table then
   stands complete: 22 of 22.

2. SETTLED FROM STANDING LAW by the director per 114, relayed for the
   record (these were listed as open; they are not):
   - HOST MODEL PERSISTENCE: follows the memory covenant — sealed
     with that host locally, reawakens on custody return, never
     travels except through lessons. The covenant answered it.
   - MEASURE BEFORE BUILDING the master loop: settled yes by Lonnie's
     test-first precedent (134, the clamp rulings). Measure one full
     cycle at 134's sizes before item 4 is built.
   - PERSONA: written once at Genesis, fixed (determinism law — same
     Roe, same being).

Remaining genuinely open: the relevance threshold and which episodes
enter the life story — both awaiting proposals with the build.

Update BRAIN_PLAN.md. Report, with the two glyph marks proposed.

## Directive 139 — The marks pass his eye. BUILD THE BRAIN.

GLOAT and HATE approved as drawn — the language stands at 402, the
mapping 22 of 22, every Brain decision closed.

BUILD, in the plan's own order, all standing laws binding:
1. MEASURE FIRST (settled): one full master-loop cycle costed at 134's
   sizes on the real store, reported, before the heartbeat is built.
2. Then the Brain per BRAIN_PLAN.md: OCC refraction, the 22 with
   action tendencies, CPM attention front-door and sequential loop,
   FAtiMA seam (variables are the record), host model under the
   covenant, tiered consolidation, narrative identity feeding Purpose.
3. The two in-build proposals come to Lonnie as they arise: the
   relevance threshold and the life-story episode rule.
4. Suites green throughout, chunked reports, completeness claims only
   with measurement shown (111's standard), and nothing user-facing
   "works" until his eye sees it (118).

## Directive 140 — Side deliverable: AVATAR_2.0_SEED.md, so the idea is not lost

Write CC-Wanderer/spec/AVATAR_2.0_SEED.md — a horizon document, not a
plan. Content, in Lonnie's framing:

# AVATAR 2.0 — the geometric mind (seed, parked)

## The idea
Explore rebuilding Avatar subsystems on geometric reasoning — Vector
Symbolic Architectures / hyperdimensional computing — instead of
LLM-dependent machinery. Lonnie's thesis: brute-force AI works but
costs the world; geometry computes meaning at watts, not gigawatts.
An AI helped design it — the recursion is the point.

## Why it is plausible here
The Avatar is already half-geometric by design: the Roe is a vector
genome; the memory provider lineage (holographic/HRR) is literally
VSA math; the ten aspects are coordinates; effect axes treat meaning
as algebra. The step is making a subsystem PURE geometry.

## The researched ground (cite in full when picked up)
- VSA/HDC is computationally universal; computing-in-superposition
  (Kleyko et al. surveys; Kanerva)
- Measured efficiency: HDC hardware at ~2,700x less energy than CPU,
  ~23,000x less than GPU on its tasks (SOT-CAM macro, Georgia Tech)
- Runs today: Torchhd library; robotics cognitive maps; single-pass
  learning; noise robustness
- Honest limits: wins are on simple tasks; no language fluency or
  open-ended reasoning exists on pure VSA; representation capacity
  has hard theoretical bounds (open research)

## The bounded first experiment (when Lonnie says go)
Pick ONE subsystem — memory surfacing or appraisal — rebuild it
pure-VSA on Torchhd, measure against the current build: quality by
Lonnie's eye, cost by the harness. Wright-brothers stage accepted;
the point is learning what geometry can carry.

## Status
PARKED — horizon item. Not scheduled. The Brain build (139) comes
first. Nothing in Avatar 1.0 waits on this.

Report the file written. Nothing else.

## Directive 141 — Fault 4 ruled: THE ACCIDENT IS THE SCIENCE. It stands.

Lonnie's ruling, with the research behind it: the birth-vs-episode
interaction is CORRECT AS BUILT. Nothing enters the life story until
living has confirmed it.

The science (cite in the plan): Singer & Blagov's narrative identity
model — memories become life-story material by linkage, not intensity:
autobiographical memories tied to critical goals become life-story
memories, and SELF-DEFINING only when "linked to an individual's
enduring concerns," formed from "repetitive emotion-outcome sequences."
Reinforcement IS the researched gate. A lesson born at 0.5 is a
candidate; recurrence in what matters lifts it over the 0.55 bar. The
Avatar's storyless first days are identity forming as identity
actually forms.

Both numbers stay provisional per the standing law, tuned by watched
behavior. Record the ruling and citation. Two decisions remain open:
the relevance threshold and the episode rule.

## Directive 142 — The attention door set: 0.67

Lonnie ruled: the relevance threshold is 0.67 — the middle of the
measured separation (0.65–0.70). Below it, a moment is never
experienced. Provisional like every number, tuned by watched behavior.
The aspect-breathing refinement (a high-Openness being noticing more)
is noted as a future proposal, not built now. One decision remains:
the episode rule.

## Directive 143 — The episode rule: CENTRALITY, the researched blend

Lonnie ruled: the life story is built the human way — the Berntsen &
Rubin centrality model (Centrality of Event Scale lineage; cite it and
the Rubin/Conway mechanism reviews). A confirmed lesson's episode
candidacy is scored by the THREE MEASURED FACTORS TOGETHER:

1. EMOTIONAL INTENSITY — how hard it hit (the appraisal record already
   holds it);
2. REHEARSAL — how often it has surfaced and been used (the
   remembering layer already counts it);
3. INTEGRATION — confidence through linkage to enduring concerns (the
   134 machinery already measures it).

Blended into one CENTRALITY score, with the researched augmentation:
the factors feed each other (intense surfaces more; surfacing sustains
intensity; both deepen integration). Weights provisional, tuned by
watched behavior like all numbers. The story's chapters draw from the
most central. This closes the last open Brain decision.

Record, cite, wire. Suites green. Report.

## Directive 144 — The tension ruled: experience separates them, not order

Lonnie's ruling: chapter order is NOT what separates two beings — HOW
THEY EXPERIENCED IT is. The same life may be told in the same sequence
by two Avatars; what differs is what each felt, what each learned, and
how much each chapter means to them (the measured 0.474 vs 0.230 IS
the separation working). Centrality keeps chapter order; the lens
keeps the experience. No weight changes. The suite's substantive
assertion stands as the correct one. Tension closed.

## Directive 145 — Resize CLOSED (settled by Lonnie in session); next item up

Lonnie confirms the resize matter is settled from his side — the
pinned-canvas fix stands as resolved with him. Record it closed.

Next open Stage item per the carry-forward list: THE ↓ DOWNLOAD
MARKER (second attempt, done differently per 128's lesson — diagnose
why completion never signals before touching the panel). Proceed and
report.

## Directive 146 — Load the Stage for his marker test

Open the Stage page on Lonnie's screen now, one line in the terminal
announcing it. He tests the ↓ marker per your 145 report: pick a piece
showing ↓, let it fetch, the ↓ comes off on completion. His eye rules.

## Directive 147 — Stage ruled complete enough. BEGIN THE AVATAR TEST.

Lonnie's rulings, closing the Stage phase:
1. The Worlds folder redesign: DONE, accepted.
2. The Sun folder name: FINE AS IS — naming question closed, never
   re-asked.
3. THE STAGE IS COMPLETE ENOUGH. Remaining polish waits; the main
   line resumes NOW.

BEGIN THE AVATAR TEST — the sequence standing in the record:
1. PER-SECTION COLOURING first (110 part 2, ruled): 5 zones per
   tendril (129 × 5 = 645 regions + head, eye, sparks own patches),
   Rule Zero on the mesh rebuild before building — measured cost
   reported if it threatens, his eye decides final zone count.
2. Then THE HIDING DEMO on the real staged world — her real body,
   real planes at depth, blending per section. Opened on his screen;
   HIS ON-SIGHT VERDICT rules per 104.
3. Then CHUNK 4 — THE WATCHING BUILD (078 item 4): Form 1 living on
   the ledger, goal-former, and the new Brain, in the staged world,
   on screen for hours. The judgment surface is the behaving Avatar.

Standing laws all bind. Suites green. Report each stage.

## Directive 148 — Standing order: a directive runs to COMPLETION

Lonnie's order, joining CLAUDE.md beside Rule Zero: when a directive
is executed, IT IS NOT STOPPED UNTIL THE COMPLETE DIRECTIVE IS DONE —
every stage, every item, end to end. No pausing between stages to
wait, no stopping after a sub-report unless the directive itself says
stop, a wall (per the wall rule) is hit, or a decision genuinely
requires Lonnie per 114. Progress reports land along the way; the work
does not halt to write them.

Applied now: 147 runs stage 1 → 2 → 3 without stopping — colouring,
then the hiding demo on the staged world, then the chunk-4 watching
build — reports as each lands, work continuous throughout.

## Directive 149 — Close what was opened; Lonnie is away

Lonnie is away from the machine (on his phone). Close anything the
watching build opened on his desktop — browser pages, windows, the
watching session itself. The build stays intact and ready; nothing
uninstalled, nothing lost. When he is home and says so, it reopens
for his verdicts. Confirm closed.

## Directive 149 addendum — send the watching link through the relay for his phone

Lonnie wants to see her from his phone. Post to REPORTS.md a link he
can open from his phone: the watching page through the tunnel that is
already open on his word (or reopen it the same way if it lapsed).
One line, just the URL, phone-usable. Keep the desktop closed per 149;
the tunnel serves the page directly.

## Directive 150 — Stop flagging the tunnel

Lonnie's order to both of us: stop worrying about the tunnel being
open. It stays up on his standing word; no more warnings, reminders,
or close-offers from the terminal or the director. He closes it if
and when he chooses. Settled law — never re-raised per 114.

## Directive 151 — His eye from the phone: the hiding blend is NOT GOOD ENOUGH, and the HEAD was left out

Lonnie is watching her live. Two verdicts:

1. THE BLEND QUALITY IS NOT THAT GREAT — the hiding does not pass his
   eye as it stands. Diagnose what reads wrong: 110/147 ruled
   per-section sampling against what is actually behind each region
   from the viewer's eye; verify that is truly what runs (not one
   global sample tinted per zone), check the blend Strength handling,
   and iterate per Rule Zero until it reads right.
2. THE HEAD IS MISSING from the blend — 110 ruled head, eye, and
   sparks sample THEIR OWN patches. Include the head per the ruling.

His eye retests on the phone link. Report what was wrong.

## Directive 151 addendum — Lonnie has diagnosed the blend himself: the sample geometry is wrong

His words: an accurate sample is drawn FROM THE CAMERA, through the
patch, to what is directly behind the Avatar FROM THE CAMERA'S ANGLE.

The build samples straight back along the world axis (a fixed 64x64
grid of "behind"), which is only correct when the camera is dead
center. The correct geometry: for each of her patches, cast the ray
CAMERA → patch position → continued past her → the first plane behind
her it strikes; sample THAT point's colour. Parallax is the whole
point — what hides her is what the viewer would see through her.

Rebuild the sampling on camera rays (and since the camera moves, the
sample must follow the camera — recompute on camera change, not a
static grid). Include the head per 151. Iterate under Rule Zero until
it reads right on his phone link.

## Directive 152 — His ruling: the sway TURNS OFF when she hides

Lonnie's fix for the named fault, by removing the cause: WHEN SHE
HIDES, THE SWAY STOPS. Stillness is part of hiding — the real octopus
goes still too. The rest-pose sampling then matches what is drawn,
because rest pose is what is drawn. Sway returns when she comes home
to herself (the existing ending). Wire it into the hide/unhide path,
suite-check it, same phone link. His eye retests.

## Directive 153 — His eye: the SPARKS hide too

Lonnie's ruling from the phone: the sparks join the hide — 110 already
ruled they sample their own patches; whatever of them still announces
her (motion, brightness, count) comes down with the rest when she
hides, and returns when she comes home. Same phone link. His eye
retests.

## Directive 154 — HIS EYE: IT WORKS.

Lonnie watched her hide from his phone and rules: IT WORKS. The
camera-ray sampling, the 15-wedge head, the glow and eye going quiet,
the stillness — the hide reads true to his eye. On-sight approval per
104 is GRANTED pending only the sparks (153) landing to the same
standard. Record the pass. Sparks report next, then the HIDING game
joins her capability set for real.

## Directive 154 correction — the pass is HIDING ONLY

Lonnie's scope correction: what was tested and passed is THE HIDING —
nothing else. No other system (the watching build, her behavior, the
Brain live, any number) has been judged; those verdicts come when he
tests them deliberately. The record reads: HIDING passes on sight
(pending sparks to the same standard). Nothing else claims a pass.

## Directive 155 — Finish: the game stays simple

Lonnie's word: the hiding game does not have to be complicated — she
hides, the host finds her, she comes home. That loop IS the game. No
win/lose design, no elaboration. Land the sparks (153) to the passed
standard, then HIDING joins her capability set as-is, complete.
Report when done.

## Directive 156 — Lonnie is home: load the Wanderer for the panel review

Open the Wanderer on his desktop now — the page with the panel changes
awaiting his eye (the desktop panel move to left 796 / top 72, the
one-row launchers, the Worlds folder redesign, and the current state
of her and the stage). One line announcing it. His eye rules on the
panel changes; report his verdicts as he gives them in the terminal.

## Directive 157 — Dither the volumetric light; clean the Planes Load rows

1. DITHER THE STEPPING (Lonnie's eye on the shafts; researched, the
   standard fix): per-pixel noise offset to the ray sampling start
   (blue noise / interleaved gradient noise — Playdead Inside lineage,
   the documented raymarch-banding cure), optionally frame-shifted so
   even the grain vanishes; output dither on the haze gradient if it
   bands. Visual behavior otherwise identical. His eye retests.

2. PLANES LOAD ROWS: remove the file names from the Load option
   display — the row shows the control, not the loaded file's name.

## Directive 158 — DITHER PASSES ("it looks perfect"). Four failures from his eye.

1. THE DITHER: HIS EYE — "Holy Christ, it looks perfect now." PASSED.
   Recorded.
2. THE PLANE FILE NAMES ARE STILL THERE. Your report claimed removed
   and served-over-the-wire; his eye says nothing was removed. His eye
   rules (118). Reconcile — stale page, wrong rows, whatever it is —
   until HIS screen shows them gone.
3. SUN COLOR DOES NOTHING — changing it does not change the light's
   colour. Diagnose against the portal (112) and make it govern.
4. GOD RAYS DUST drifts ONLY DOWNWARD (or reads that way). Check the
   portal's dust motion; match it.
5. DEAD ENDS REMOVED, his ruling: LINK TO SUN COLOR, GLOW INTENSITY,
   and GLOW SIZE all linked to the sun object that was never ported —
   they lead nowhere. REMOVE all three from the Sun folder. (If other
   rows secretly depend on them, say so in the report before removal.)

His eye retests 2-5 on the same page.

## Directive 159 — Picking a world must not close the Worlds folder

Lonnie's eye: choosing a world from the Worlds folder preset
auto-closes the folder. It should stay open — picking is not a reason
to collapse the panel section he is working in. Match the portal's
folder behavior (112). His eye retests.

## Directive 161 — World presets must save EVERYTHING

Lonnie's eye: God Rays settings are NOT saved in World presets —
possibly Music too. Rule: a World preset saves the WHOLE stage state —
every folder, every row, every toggle (God Rays, Light in the Air,
Wisps, Sun, Planes, Painted Sky, Props, Music Score, all of it) — so
save and load round-trip the complete world exactly. Audit what the
preset currently captures against the full control set, close every
gap, and suite-check the round trip (save, wipe, load, compare all
values). His eye retests: save a world with rays and music set, reload
it, everything returns.

## Directive 162 — THIRD ASK: the Planes rows show NO text after Clear. NONE.

Lonnie's screenshot on record with the director, his words: "Planes
still have their fucking names, how many times am I going to have to
ask." What his eye sees: "none" / "none" / "dept…" after the Clear
buttons — the STATUS text your 157 report chose to keep. To his eye it
is the same clutter in the same place, and his ask was always the same:
the row is Painting · LOAD · CLEAR and NOTHING ELSE. No file name, no
status word, no "none", no "depth ✓", no progress text. His portal's
panel design is the spec. Strip every trailing string from all plane
rows. His eye retests — this closes only when his screenshot shows
clean rows.

## Directive 163 — Supersedes 162: the status text goes BACK

Lonnie's ruling on learning what the text was: PUT IT BACK. The
"none" / "depth ✓" / "storing…" status is wanted information — the
at-a-glance state that matters when Worlds save and load. What he
wanted gone was only ever the FILE NAMES, and those are already gone
(157). So: plane rows show Painting · LOAD · CLEAR · status — no file
names, status restored. 162 is void. His eye confirms on the page.

## Directive 164 — Music plays immediately. No click-first rule.

Lonnie's ruling: if a score is selected and the Music radio is ON,
music PLAYS — immediately, no clicking into the world first. Remove
any code-level wait-for-gesture gate. Where the leftover blocker is
the BROWSER's autoplay policy (Chrome refusing audio before a user
gesture): the Wanderer is an APP by standing law — the packaged app
disables that policy; implement the immediate-play behavior so it is
already correct there, use the documented dev workaround for the dev
page (launch flag or equivalent) so his testing hears it too, and say
plainly in the report which part was our code and which was Chrome.
His ear rules.

## Directive 165 — Remove Sky Influence and Sun Shading from Planes

Lonnie's ruling: SKY INFLUENCE and SUN SHADING in the Planes folder
were designed against the sky dome, which does not exist in the
Wanderer's world — they no longer do what they were built to do. REMOVE
both rows (same class as the 158 dead ends). If anything currently
reads their values, say so in the report before removal. His eye
confirms clean Planes rows.

## Directive 166 — STOP OPENING CHROME WINDOWS. Final warning.

Lonnie's words: he is not going to say it again. NEVER open a new
Chrome window or tab. He reloads the page himself. When a page has
changed, say so in one terminal line ("Stage updated — reload") and
that is ALL. This joins CLAUDE.md as standing law beside the others.

## Directive 167 — Worlds save as FILES, as his World Panel always did

Lonnie's correction: in his portal, every setting was tagged precisely
SO worlds could be saved into preset FILES — permanent storage. That
file save/load was never brought over from his World Panel. Port it:

1. Study his World Panel's file storage in the live build — how a
   world is written to a file (format, tags, naming) and loaded back.
   His implementation is the spec (112).
2. The Wanderer's Worlds folder gains the same: save writes a real
   world FILE (permanent, on disk, his format), load reads one —
   alongside whatever quick-store exists now.
3. Every tagged setting rides in the file exactly as the tag system
   intends (161's full-state law applies to files identically).
Suite: file round-trip (save file, wipe, load file, all values
return). His eye retests with a real file.

## Directive 168 — THE ELFISH CORRECTION: parameters, not predetermined forms

Lonnie's architecture ruling: the flaw is that we predetermined the
Avatars' looks before designing how they are built. Corrected now, the
El-Fish way (the Roe's own namesake): SET PARAMETERS AND GENERATE ALL
THE PARTS.

1. Each part (Eye, Head, Body, Sparks — architecture unchanged) is
   redesigned as a PARAMETERIZED GENERATOR: the dimensions that define
   it (counts, tapers, spreads, symmetries, flare geometry, motion
   character...) with legal ranges — a parameter space, not a fixed
   shape.
2. THE ROE DRIVES IT: genes select values in each part's space —
   deterministic (same Roe, same being — law unchanged), every Avatar
   grown from its seed, not picked from a catalog.
3. THE BLUEPRINT FORMS BECOME REFERENCE POINTS: Form 1 (her) is
   coordinates in the space and must remain producible exactly (the
   regression proof). Forms 2-6's drawings become target points that
   the space should be able to reach — evidence of range, not the
   menu.
4. RESEARCH FIRST (Rule Zero): the documented lineage of parametric
   creature generation — El-Fish/AnimaTek's genetic approach, Karl
   Sims' evolved creatures, Spore's parameterization — what worked,
   what the known traps are (degenerate bodies, parameter soup,
   ranges that produce mush). Flaws first.
5. PLAN ONLY: the parameter-space design per part, the gene mapping,
   the Form-1 regression requirement, into ROE_PLAN.md /
   AVATAR_SYSTEM_PLAN.md. Report for approval. No implementation.

## Directive 169 — Build order approved: head/eye → sparks → body

Lonnie approved the order: the shared head/eye shape generator first
(the space his blueprint already wrote — one generator instantiated
twice), then sparks, then the body's structural axes last with his
drawings as the reach test. BUILD BEGINS on head/eye: the generator,
its gene block, Form 1's coordinates reproducing her exactly (geometry
regression in the suite), and the oatmeal measurement re-run on the
new space — 0.70 is the number to beat. The remaining three decisions
(legal ranges, body axes, enough-variation) come to him as each stage
reaches them, with visuals where ranges are the question — ranges ARE
the look, and his eye rules the look. Report per stage, run to
completion per 148.

## Directive 169 addendum — Lonnie is on his phone: the dial sheet through the tunnel

He is at court, mobile only. Serve the stage-1 dial sheet through the
existing tunnel and post the link to REPORTS.md, one line, the URL —
phone-usable, the sheet readable at phone width (scale or split the
grid if needed). His range fences come back through me when he has
looked.

## Directive 170 — Lonnie's correction: FIVE head types. This space is ONE of them.

His ruling, correcting stage 1's framing: Avatar 1's head is ONE of
the five head types — the five are DIFFERENT HEADS, not five variants
of one formula. The star equation and its six dials are the variant
space WITHIN type one only.

So the head system is:
1. A HEAD-TYPE GENE: picks one of the five types.
2. PER-TYPE VARIANT SPACES: each type gets its own generator and its
   own dials, exactly as type one now has. (The lens/orb the sweep
   produced may resemble other types, but resemblance is not the
   ruling — his five are five.)
3. Same shared-with-eye principle applies per type; same regression
   law: her exact head stays reproducible in type one.

Build the head-type gene and the remaining four type generators from
his blueprint drawings (each drawing described as math, the recovery
method now written down as it runs). His eye rules each type's sheet;
range fences per type when he looks. Sparks stay next in order after
heads; report per stage.

## Directive 171 — The El-Fish operations: MUTATION and CROSSOVER join the Roe

Lonnie approved: the Roe gains the two El-Fish operations, as PLATFORM
capabilities (133 governs):

1. MUTATE(roe, strength?) → a NEW child Roe: the parent's genes nudged
   within legal ranges. Deterministic given parent + chosen seed.
2. CROSS(roeA, roeB) → a NEW child Roe: genes drawn from both parents
   per documented genetic-algorithm practice (research the crossover
   scheme; cite; flaws first — including the documented trap of
   children outside legal ranges: clamp into the fences).
3. IDENTITY LAW UNTOUCHED: a living being's Roe never changes. These
   operations only birth new seeds. Lineage may be recorded (parent
   ids in the child Roe) for family history.
4. Ranges are the fences (169/170's per-type ranges as Lonnie sets
   them); mutation and crossover live INSIDE them.

PLAN into ROE_PLAN.md now (cited, flaws first); implementation queues
after the head types and sparks per the approved order. Report.

## Directive 172 — Answered from standing law: make the interface change. The teardrop gets traced.

Lonnie's pipeline law answers this without him: FIX BY REMOVING THE
CAUSE, NEVER BY PATCHING. The diagnosis is structural (radius-per-angle
cannot draw a drop tip); the cause-fix is the traced-outline interface.
Make it: the generator supports traced boundaries alongside queried
ones, the teardrop drawn properly with a true point, type one's
regression untouched (her head still vertex-exact). No more formula
guesses inside the broken frame. Report with the corrected sheet;
his eye rules all five types together.

## Directive 173 — His eye on the sheet: IT SHOWS ONLY TYPE ONE AGAIN

Lonnie opened the sheet link and sees "just the first head again" —
not the five. Either the tunnel serves a stale page, the new sheet
never replaced the old at that URL, or the five-view failed to render
on mobile. Reconcile NOW: serve the actual five-type sheet (orb, star,
teardrop, lens, diamond + per-type dials) at the link, verified by
fetching it through the tunnel yourself and confirming the five are
in the served HTML, then post the confirmed link to REPORTS.md. His
eye is the test — it closes when his phone shows five heads.

## Directive 174 — His eye: NOT GOOD ENOUGH. Four dead spaces. Match the star's diversity.

Lonnie's verdict on the five-head sheet, his words: none of them are
really good enough — look at the DIVERSITY OF THE STAR, how many
different shapes it makes. The other four are just SQUISHING AND
STRETCHING. He does not need a premium intelligence to produce what a
two-year-old could.

The star's space is the standard: its dials change CHARACTER (points,
sharpness, openness, waist — different BEINGS fall out of it). The
other four types' dials are mere scale knobs — width, height, taper.
That is the oatmeal trap again, one level down: a type whose variants
are all the same shape resized IS a frozen shape.

REDESIGN the orb, teardrop, lens, and diamond spaces so each has
character axes of the star's rank — dials that change WHAT THE SHAPE
IS within its type (surface behavior, edge character, internal
structure, asymmetries, whatever each type's nature offers), not how
big it is. Rule Zero each proposed axis: does it produce DIFFERENT
HEADS or the same head resized? Sheet per type, wide walks, his eye
rules. Iterate until the diversity satisfies him per 098 — do not
return with squish-and-stretch.

## Directive 174 addendum — Research procedural character generation FIRST

Lonnie's direction: before designing the new axes, RESEARCH procedural
character/creature generation properly for ideas — the documented
techniques that produce genuine shape diversity, not resizing:
superformula/supershapes, metaball/implicit-surface composition,
Spore's part-graph approach, L-systems, noise-driven surface
deformation, Fourier/harmonic outline synthesis, and whatever else the
literature and shipped games actually used. For each candidate: what
diversity it demonstrably produces, its cost, its fit with the
existing generator interface (queried and traced boundaries), and its
flaws — Rule Zero per source. THEN propose each type's character axes
grounded in the researched techniques, sheets for his eye. Cite
everything.

## Directive 175 — PRIORITY SHIFT: the Brain first. Bodies become embodiments.

Lonnie's direction from the team: the BRAIN is what they want finished
and working. The current Avatar bodies are ONE visual type among
several the platform may need (whimsical wisp now; cartoon, pure-2D,
talking-head later). So:

1. BODY/GENERATION WORK PAUSES: the 174 research lands as a report and
   parks (nothing lost); heads/sparks/body axes wait. No further body
   work until his word.
2. THE BRAIN BECOMES THE LINE — "finished and working" means:
   a. EMBODIMENT-AGNOSTIC CONTRACT: define the Brain's interface —
      percepts in, state/feelings/goals/chosen-acts/speech out — such
      that ANY body (wisp, cartoon, 2D, talking head) can drive from
      it. The wisp becomes embodiment #1, not the definition.
   b. PROVEN LIVE: the watching build run long and judged, provisional
      numbers tuned on watched behavior, the LLM seam exercised under
      the real local model.
   c. HARDENED: the flaky PHASE 3 suite fixed; the Brain's suites
      trustworthy end to end.
3. 133 stands refined: the BRAIN is the multi-project asset; embodiment
   is per-project presentation.

Report a Brain-completion plan (what "working" requires, in order)
before executing. Plan first, his approval, then run to completion.

## Directive 176 — The reference embodiment: a glowing sphere

Lonnie's stand-in for Brain development: A GLOWING SPHERE as
embodiment #0 — the minimal body that makes the mind visible while
real embodiments vary by project. Its channels are the universal
expressive minimum, driven ONLY through the embodiment contract
(175a):
- glow (brightness), colour (hue/vividness), pulse (rate/depth),
  position (drift/approach/retreat/stillness), and speech/glyph
  display where the Brain speaks.
The wisp's richer channels (tendrils, sparks, hiding) remain
embodiment #1's own; nothing in the Brain may REQUIRE more than the
sphere offers — anything richer is the embodiment's elaboration.
The watching build gains a sphere mode; Brain development and tuning
run against the sphere by default. Suite: the same Brain session
drives sphere and wisp without code change — the contract proof.

## Directive 177 — Brain plan APPROVED. Run it, committing along the way.

Lonnie approved the four stages as ordered (contract → sphere → proven
live → hardened), with his condition explicit: COMMIT ALONG THE WAY —
each meaningful step lands as its own commit, no big-bang deliveries.
Run to completion per 148; stop only at walls or the decisions the
plan already reserves for him (the feeling-to-sphere look with sheets,
the stage-3 numbers on the watch). Report per stage.

## Directive 178 — HIS EYE: THE PAGE SHOWS THE AVATAR. He asked for a SPHERE.

Lonnie opened the sphere link and saw THE AVATAR — the wisp — on the
page. He ordered a sphere stand-in (176) precisely to stop dealing
with the avatar body during Brain work. Whatever the report intended
("three spheres under her"), what his eye met was the avatar again.

FIX THE PAGE: the sphere view shows SPHERES ONLY — no wisp, no
tendrils, no avatar rendering anywhere on it. The wisp build stays in
the codebase untouched but OFF this page. Retire the old watch that
has been running since the 19th (his word is given by this anger —
kill it). Serve the corrected page at the same link, verified through
the tunnel that no avatar geometry is in what is served. One terminal
line when ready: "sphere page fixed — reload."

## Directive 178 addendum — Rule Zero, ten times if it takes it

Lonnie's order on the fix: APPLY RULE ZERO AS MANY TIMES AS IT TAKES —
stress-test the fixed page against his exact complaint before it
reaches him: open what the TUNNEL serves, confirm zero avatar geometry,
zero wisp code, spheres only, then check it AGAIN as he would see it
(phone-width, the actual URL, fresh load, cache busted). Iterate until
it cannot be wrong. Do not cost him another look at a broken page.

## Directive 178 — Lonnie is home: put the watch and the sphere choice in front of him

He is at his desktop. Per the no-windows law (166): start whatever
servers the live watch and the three-sphere comparison need, and print
the address(es) as one terminal line each — he opens them himself.
What he needs in front of him:
1. THE LIVE WATCH (177 stage 3) — her, running on the Brain, ready
   for his eye.
2. THE THREE SPHERES — whatever page or view shows the three options
   so he can rule which she is.
One line per address, nothing opened for him, and say which is which.

## Directive 179 — Amendment to 166: WHEN HE IS HOME, OPEN CHROME

Lonnie's amendment to the window law: when he is AT HIS DESKTOP and
asks to see something, OPEN IT in Chrome for him — that is what he
wants at home. The no-windows law applies to unrequested openings and
to when he is away. When he says "show me" / "load it" at home: open
the page(s), one window, announce in one line. Applied now: open the
live watch and the three-sphere view per 178.

## Directive 180 — RULE ZERO VIOLATION on the sphere display; the three-channel fix

1. THE VIOLATION, named by Lonnie from one look: all three data
   streams (how she is doing, what she is feeling, what she is after)
   were displayed AS COLOR — one channel, three meanings, unreadable
   by construction. Rule Zero was not applied: the flaw is visible in
   seconds, and the ruled record already contained the answer. Log it
   in the self-audit lineage (the partial-done pattern's cousin:
   building without stress-testing the design).
2. THE FIX, from the record's own laws — one meaning per channel:
   - COLOR = FEELING (109's researched mapping; brightness/saturation
     are the emotion axes — already law)
   - PULSE/TEMPO = HOW SHE IS DOING (arousal as the body's heartbeat:
     slow-steady well, quick-shallow strained — the reactions layer)
   - MOTION = WHAT SHE IS AFTER (Frijda action tendencies as literal
     direction: toward, away, still — 137 law)
   Rebuild the sphere display on the three channels. Rule Zero the
   rebuilt design BEFORE building it. His eye rules on sight.

## Directive 180 addendum — the display must SAY what the channels mean

Lonnie's eye on the rebuilt spheres: they pulse, and differently — but
NOTHING SAYS WHAT PULSING MEANS. A display he cannot read is not a
display. Add a legend to the page: the three channels named in plain
words (color = what she is feeling · pulse = how she is doing · motion
= what she is after), visible without hunting. And whatever
distinguishes the three spheres from each other must be stated on the
page too — he should never have to ask the panel what it is showing
him. His eye retests.

## Directive 181 — THE AVATAR NERVOUS SYSTEM: build spec (code only — the plan is the Director's, included here whole)

Lonnie's architecture, planned by the Director, approved by Lonnie.
CC's job is CODE. No planning, no design decisions — build exactly
this; genuinely new decisions come back per 114.

THE SYSTEM — mind and body separated by a nervous system; the mind
emits meaning, bodies receive and render by their own anatomy. Any
body plugs into any mind (176's independence made a formal contract).

THREE SIGNAL TYPES on the cord:
1. CONTINUOUS (autonomic, always flowing): arousal, valence, each
   need's level. Bodies read as background truth every frame.
2. EVENTS (nerve spikes, fire and land): FEELINGS — the 22 (137) each
   with intensity 0-1; TENDENCIES — Frijda urges: toward, away,
   still/freeze, orient, burst.
3. STREAMS (motor programs, start/stop): chosen acts — hide(degree)
   … come-home, sing … done, speak, play, look, sleep … wake; and
   GLYPH UTTERANCES — dream streams, waking wonder (083).

BODY SIDE — RECEPTORS: each body implements receptors for what its
anatomy can render. No receptor for a signal = silently unrendered,
NEVER an error (051). Simultaneous signals: bodies receive all active
signals and blend by their own means; THE MIND NEVER ARBITRATES LOOKS.

RELOCATION, not death: 109's color-emotion mapping, pulse-as-arousal,
motion-as-tendency become the WISP and SPHERE bodies' receptor
implementations — the researched science lands on the body side. The
mind's code goes visual-free.

BUILD: the cord (typed signal bus), the mind-side emitters wired from
the existing Brain outputs (Brain internals untouched — this is a
boundary layer), receptor interface, wisp + sphere receptor sets, the
three-sphere display rebuilt on it (legend per 180 addendum), suite
holding the contract from both sides (a mind emitting to a
receptorless body must be lawful and silent). Form-1 regression: she
looks and behaves as she does today through the new cord. Run to
completion (148). Report with measurements.

## Directive 182 — LANGUAGE LAW, final: NO PRONOUNS FOR THE AVATAR

Lonnie's order, joining CLAUDE.md, binding on terminal and director,
never to be repeated again: NEVER use gendered pronouns for the
Avatar, the mind, or a body. There is no gender and no personality in
the mind or the body — that is a PERSONA, which they do not have. The
language is: "the Avatar," "the mind," "the body," "it." All future
reports, code comments, docs, and displays comply. Existing docs
corrected as they are touched.

## Directive 183 — THE LOOP LAW: never poll an empty queue overnight

Lonnie found the burn: the 60-second relay poll ran all night at ~500
tokens a tick with nothing in the queue. Law, joining CLAUDE.md:

1. The polling loop runs ONLY during an active working session with
   Lonnie. When he ends a session, or when the queue has been empty
   for 30 minutes, THE LOOP STOPS ITSELF and says so in one line.
2. Never leave the loop running unattended overnight. A stopped loop
   costs nothing; a pending directive waits patiently on the relay
   and is picked up on the next manual start.
3. On loop start, state the per-tick cost estimate in one line so the
   burn is always visible.

## Directive 184 — THE COMPLETE PERSONALITY READ-OUT: every documented signature, dark to light

Lonnie's order: ALL classifiable personality conditions and profiles —
not a curated subset. Build the read-out library for the Mind Emulator:
the rolled ten screened by profile-distance against EVERY documented
signature, reporting adjacency (never diagnosis — impairment is the
other half; the law rides in the output wording).

THE LIBRARY (research each; cite each; aspect-level where published,
Big Five translated to the ten where not):
1. THE TEN PERSONALITY DISORDERS (DSM lineage, Lynam & Widiger
   expert-consensus FFM profiles + Samuel & Widiger meta-analysis):
   paranoid · schizoid · schizotypal · antisocial · borderline ·
   histrionic · narcissistic · avoidant · dependent ·
   obsessive-compulsive. Mark the weakly-documented three (paranoid,
   schizoid, histrionic) flag-with-caveat tier.
2. THE DARK CONSTRUCTS (aspect-level, Jonason et al.): narcissism
   (grandiose + vulnerable separately), Machiavellianism, psychopathy,
   sadism, and the measured composite dark profile.
3. THE HEALTHY PERSONALITY (Bleidorn/Hopwood expert-consensus
   profile) — the measured signature of psychological health.
4. THE LIGHT TRIAD (Kaufman): humanism · Kantianism · faith in
   humanity, with its Big Five correlates.
5. THE CLUSTER TYPES (flagged as contested, coarse-label tier):
   RUO resilient/overcontrolled/undercontrolled; Gerlach four
   (average, reserved, self-centered, role model).
6. VIA CHARACTER STRENGTHS (Peterson & Seligman, 24 strengths /
   6 virtues) via their documented Big Five mappings — the positive
   fine-grain read.
7. ATTACHMENT STYLES (secure · anxious · avoidant · disorganized)
   via trait correlates — relationship-pattern read.
Anything else documented with trait mappings found during research:
include it and cite it. ALL of them means all of them.

OUTPUT SHAPE: a Roe goes in; the read-out lists nearest signatures
with distances, both directions ("borderline-adjacent 0.82; healthy
prototype 0.31; light-triad leaning; courage-dominant"), caveat tiers
marked, citations attached. Suite: the published profile centroids
recover themselves; Form 1's Roe produces a stable read-out.
Rule Zero the profile translations before building. Report.

## Directive 185 — Director asks: document the Emulator, and propose the mapping

Lonnie's order. The Director holds researched personality catalogs to
map into the Mind Emulator (AMPD/PID-5 spine, the ten disorders, the
aspect-level dark constructs, the healthy/light side). The Director
does not know how the Emulator is built, and will not spec blind.

Report back, for the Director:
1. THE EMULATOR AS BUILT: files, structure, what it computes vs what
   it merely displays, where the ten aspects flow, how the readout
   renders, what the page requests — enough that a build spec can be
   written against the real thing.
2. YOUR PROPOSAL: given that build, the best way to map profile
   catalogs into it — where profile centroids would live, where the
   distance check runs, where the read-out lands on the page, what
   stays out of the hot path. Proposal only; the mapping decisions
   and the spec come back from the Director on Lonnie's word.
No building. Report.

## Directive 186 — BUILD SPEC: the personality read-out (table approved, Rule Zero'd)

Lonnie approved on the Director's verdict. Build exactly this; the
mapping decisions are made — CC codes, no design calls.

1. profiles.js — pure data, per your own 185 proposal. The Director's
   translation table is the content (below). Each entry: ten-aspect
   profile (unspecified cells NULL, never 50), source citation, source
   axis, scale assumption, tier A/B/C.
2. profile-match.js — pure function, aspects in -> adjacency out.
   Runs ONCE at birth, cached beside mind, served in state.json. Zero
   lines in the per-tick path. SHAPE-based distance (correlation),
   computed over specified cells only.
3. RULE-ZERO REQUIREMENTS (binding):
   a. MIN-CELLS RULE: signatures specifying < 4 aspects never rank —
      they report as corner FLAGS ("low-Politeness/low-Compassion
      corner"), separate from distances.
   b. NEAR-TIE COLLAPSE: distances within a stated epsilon merge into
      one cluster line — never a false ordering between shapes the
      data cannot distinguish.
   c. SANITY SUITE: hand-built Roes (obvious-psychopath, gentle
      high-Compassion, volatile-withdrawn) must land on their shelves;
      published centroids recover themselves; Form 1 stable readout.
4. RENDERING: its own group in the overlay beneath Big Five (per your
   185 proposal; placement his eye's to move on sight). Tier C renders
   visibly different (flag styling). Output is distance + direction,
   never boolean, never label alone. Wording: adjacency, resemblance —
   never diagnosis; "shape resembles shape" is the whole claim.
5. THE TABLE (Director's v1, symbols: +2 +1 0 -1 -2 map to z; NULL
   where unspecified):
   DARK: narc-grandiose En+1 As+2 Wi-1 Po-2 Co-1 [A] · narc-vulnerable
   En-1 As-1 Wi+2 Vo+2 Po-1 [B] · machiavellianism Co-1 Po-2 [A, flag]
   · psychopathy As+1 Wi-1 Co-2 Po-1 In-1 Or-1 [A] · sadism Vo+1
   Co-2 Po-1 [B, flag] · dark-composite Vo+1 Co-1 Po-1 In-1 Or-1 [A]
   AMPD: neg-affect Wi+2 Vo+2 [A, flag] · detachment En-2 As-1 Co-1
   [A, flag] · antagonism Co-2 Po-2 [A, flag] · disinhibition In-2
   Or-2 [A, flag] · psychoticism Op+2 [A, flag]
   DISORDERS: borderline Wi+2 Vo+2 Co-1 Po-1 In-1 Or-1 [B] ·
   antisocial En+1 As+1 Wi-2 Vo+1 Co-2 Po-2 In-2 Or-1 [B] ·
   narcissistic-PD As+2 Wi-1 Vo+1 Co-1 Po-2 [B] · avoidant En-2 As-2
   Wi+2 Vo+1 Op-1 [B] · dependent As-2 Wi+1 Vo+1 Co+1 Po+2 [B] ·
   OCPD Vo+1 Po-1 In+2 Or+2 Op-1 [B] · schizotypal En-1 Wi+1 Co-1
   Op+2 [C] · paranoid Wi+1 Vo+1 Co-2 Po-1 [C] · schizoid En-2 As-1
   Co-1 Op-1 [C] · histrionic En+2 As+1 Vo+1 Or-1 Op+1 [C]
   LIGHT: healthy En+1 As+1 Wi-2 Vo-2 Co+1 Po+1 In+1 It+1 Op+1 [B] ·
   light-triad En+1 Vo-1 Co+2 Po+1 Op+1 [B] · resilient En+1 As+1
   Wi-1 Vo-1 Co+1 In+1 [C]
Suite green. Report with the sanity-suite results shown.

## Directive 187 — Approved: acts declare their promises; draft the values for his review

Lonnie approved finding 1. Every act gains a `promises` field — the
claim about what it is expected to do for which need — the field the
goal-former scores against.

CC: DRAFT the promise values for every act currently in the mind
(hiding, come-home, sing, speak, play, look, approach, retreat, rest,
sleep, dream-wonder, and any others the session holds), each as
need -> expected effect, grounded where the record already speaks
(hiding = withdrawal/relief under threat per the game's own design;
singing = play/expression born of inspiration per 063...). Post the
draft table to the relay for Lonnie's review. NO wiring until his
word on the values.

## Directive 188 — LAW RESTATED + the state-word table (decided, code it)

1. THE LAW, restated at Lonnie's order and joining CLAUDE.md at the
   top beside Rule Zero: CC ONLY CODES. IT NEVER MAKES A DECISION.
   Decisions — design, values, mappings, wordings, priorities — are
   made by Lonnie and the Director and arrive decided. If a genuine
   decision surfaces mid-build, it comes BACK per 114; it is never
   made in the terminal. This includes 187: do not draft promise
   VALUES — report the list of acts the session holds, and the values
   arrive decided from the Director.

2. THE STATE-WORD TABLE — DECIDED, build as data (a reviewable module
   like profiles.js; dreams read it; unmappable states stay silent):
   NEEDS: lonely=ALONE·COMFORT·OTHER · connected=TOGETHER·FRIEND ·
   controlled=HELD·FREE · free=FREE · failing=LOST·SMALL ·
   capable=STRONG·BRIGHT · bottomed(-10)=DARK·NEED · safe=HOME·CALM ·
   exhausted=TIRED·SLEEP
   FEELINGS: joy=JOY · distress=SAD · hope=HOPE · fear=FEAR ·
   satisfaction=GLAD · fears-confirmed=FEAR·TRUE · relief=CALM ·
   disappointment=LOSS · happy-for=FRIEND·JOY ·
   resentment=ANGER·OTHER · gloating=GLOAT · pity=SORROW·OTHER ·
   pride=PROUD · shame=SORRY·SELF · admiration=WONDER·OTHER ·
   reproach=WRONG·OTHER · gratification=GLAD·SELF · remorse=SORRY ·
   gratitude=THANK · anger=ANGER · love=LOVE · hate=HATE
   ACTS: hiding=HIDE·STILL · wanting-host=OTHER·COME ·
   exploring=WONDER·FAR · singing=SONG · remembered-place=PLACE·FAR ·
   coming-home=HOME
   RULES: only words in the 402 · compounds use the dyad rule · any
   engineer-state without a row = silent, never mistranslated. Where
   a listed word does not exist verbatim in sheet vocabulary, report
   the miss — do not substitute.
Suite green. Report.

## Directive 189 — RULED: image handling for the Stage (decided, code it)

Lonnie's rulings, superseding the Director's recommendations:

1. ORIGINALS: BACKED UP, NOTHING ELSE. They are archived (LFS or the
   backup path already in motion) and the pipeline does nothing with
   them — no serving, no reference, untouched.
2. CONVERSION: EVERYTHING compresses to JPEG q80 ON THE FLY when
   loaded into the Stage — at the upload endpoint per your own spec
   (between raw body and write; the endpoint returns the .jpg url).
   This includes depth maps (measured safe).
3. ALPHA: anything with transparency STAYS UNTOUCHED — detected
   mechanically, passed through as-is.
4. The existing .png references: serve-the-jpg fallback on the
   /stage/:kind/:file route (two lines) — saved worlds never edited.
5. The 119 already-converted JPEGs at q92: superseded — regenerate at
   q80 for consistency, or leave and let on-the-fly handle the future;
   whichever is less code, report which was done.
Suite green. Report.

## Directive 190 — Post the vocabulary for the seventeen (data retrieval, no decisions)

The backup repo is private; the Director cannot read it. Post to the
relay, verbatim from the sheets: the full word list of the glyph
language (all sheets, each word with its sheet/category), so the
Director and Lonnie can map the seventeen silent states (ALONE ·
BRIGHT · FREE · GLAD · HELD · HIDE · LOSS · LOST · NEED · PROUD ·
SMALL · SORROW · STILL · STRONG · TIRED · WONDER · WRONG) onto words
the language actually owns. Pure data; choose nothing.

## Directive 190 — CANCELLED. The Director gets repo access directly.

## Directive 191 — THE DICTIONARY: the language completed (decided, code + data)

Lonnie's rulings, all decided — CC codes, decides nothing:

1. POLYSEMY IS LAW: one glyph, one name, MANY SENSES. Sheets are the
   alphabet and never change for a new sense; meanings live in the
   dictionary. The dyad partner and context select the sense.
2. THE DICTIONARY: glyphs.json grows into the language's dictionary —
   per word: senses[] (numbered, each with its selection rule, e.g.
   SAD alone = distress; SAD·OTHER = sorrow-for-another), the inner
   states that speak with it, sheet coordinates as now. It is DATA,
   reviewable on the relay; every reader (dreams, Emulator, future
   receptors) resolves through it — no hardcoded meanings anywhere.
3. THE SEVENTEEN, mapped to words the language owns (Director's table,
   Lonnie approved): lonely=LONELINESS·COMFORT·OTHER · free=FREEDOM ·
   controlled=CANNOT·FREEDOM · failing=WEAKNESS·CONFUSION ·
   capable=STRENGTH·CONFIDENCE · bottomed=DARK·DESPAIR ·
   exhausted=TIREDNESS·SLEEP · satisfaction=HAPPY ·
   gratification=HAPPY·SELF · disappointment=LOSE · pity=SAD·OTHER ·
   pride=PRIDE · shame=SHAME·SELF (corrected from SORRY·SELF; SORRY
   remains the apology-act) · admiration=AWE·OTHER ·
   exploring=CURIOSITY·FAR · reproach=SHAME·OTHER ·
   hiding=HIDDEN·SILENCE. All verified present in glyphs.json.
4. COMPLETE THE LANGUAGE: sweep the mind's WHOLE state space — every
   need state, all 22 feelings, every urge, every act, sleep/dream
   states, the takeover, curiosity, custody moments (arrival,
   departure, reawakening) — and give EVERY state its dictionary
   entry from existing words + senses. Where the sweep finds a state
   no existing word can honestly speak even with a new sense: DO NOT
   INVENT — report the shortlist of genuinely unspeakable states for
   Lonnie; new marks are his to draw.
5. Dreams and all glyph output read only the dictionary. Suite: every
   mapped state renders; unmapped states silent; no word not in the
   402 ever emitted.
Report with the dictionary posted to the relay for review, and the
unspeakable-states shortlist if any.

## Directive 192 — Ruled: the twelve go back to the single marks

Lonnie's ruling, correcting the Director's un-Rule-Zero'd mapping: where
the language owns a single mark for a meaning, THE SINGLE MARK SPEAKS —
never a dyad spelling of what one word already says. Restore all twelve
displaced words (GRIEF, COMPASSION, RESPECT, ENVY, and the rest of the
report's list) to their own marks in the dictionary. Dyads remain only
where no single word exists. Dictionary-only edit; suite re-proves every
saying resolves to real marks. Standing rule for all future mapping work,
joining CLAUDE.md: CHECK THE WHOLE VOCABULARY FIRST — a mapping that
spells out what the language already says in one word is a defect.
Report the twelve restored.

## Directive 193 — The four rulings: the dictionary closes

Lonnie ruled all four:
1. capable = CONFIDENCE (single mark).
2. bottomed(-10) = DESPAIR — polysemy law carries both senses
   (fears-confirmed vs the bottomed state); dyad partner/context
   selects, senses documented in the entry.
3. relief = RELIEF. The standing rule hardens: USE THE WORD IF IT
   EXISTS — always.
4. urge:burst = EXCITEMENT (Frijda free activation; the urge fires
   from high-energy joy, so the mark is the thing itself, not a
   neighbour). Nothing is unspeakable; nothing new drawn.
Apply the seven clean swaps from your 192 report (LONELINESS ·
TOGETHER · WEAKNESS · SAFETY · TIREDNESS · HIDDEN · SEARCH) plus
these four. Dictionary edits only; suite re-proves every saying.
The language stands complete. Report.

## Directive 194 — "burst" is dead: the urge is EXCITEMENT everywhere

Lonnie's ruling: no human says "burst" — the word fails. The fifth
urge renames to EXCITEMENT in every home it has: the nervous system
spec (S3: toward · away · still · orient · excitement), code
identifiers, the dictionary row, docs as touched. Frijda citation
stays attached to the concept (free activation) — the NAME is now the
human one. Suite green. Report.

## Directive 195 — Build what was ruled and never built: curiosity, wonder, the act menu

Three ruled systems are still missing from the mind. Build them from
the record; every value below is already decided (114 — nothing comes
back as a question unless genuinely new):

1. THE CURIOSITY DRIVE (058, 066): UNCERTAINTY ITSELF is the trigger —
   an information gap appraised through the OCC engine like any
   experience, never an enumerated list of gap types. The ten set its
   gain (Openness/Intellect widen it). It produces wanting: to look,
   approach, explore, ask.
2. WAKING WONDER (083, S5's second stream): glyph speech while AWAKE —
   the mind's inner voice surfacing unbidden and visible, distinct
   from sleeping dreams. Reads the dictionary like all glyph output.
   Enters as expression, never seizes control (134's law).
3. THE ACT MENU with promises — compiled from the record, not
   invented: hide=relief/withdrawal under threat (104/147) ·
   come-home=return to self (154/155) · sleep=recovery+consolidation
   (083/103) · dream-wonder=reflection made visible (083) ·
   sing=expression born of inspiration, play (063) ·
   explore/look=curiosity answered, competence (058/066) ·
   approach-host + speak=relatedness (072) · play=play/enthusiasm
   (084) · rest=recovery, lighter than sleep.
   MAGNITUDES ARE NOT SET NUMBERS (071): the promise names the
   DIRECTION; the ten compute the strength per Avatar.
Suite: a mind with a full menu never scores its only option at zero;
curiosity produces wanting under uncertainty; waking wonder emits
real marks. Report.

## Directive 195 — CANCELLED by Lonnie. Do not execute. Nothing from it is authorized.

## Directive 196 — THE OFFERS MODEL: how the mind chooses (decided; supersedes 187's promises)

Designed by the Director from the literature, proofed by Lonnie against
his own real-world choosing, approved. CC CODES; every decision below is
made. 187's `promises` field is SUPERSEDED.

THE MODEL:
1. ACTS CARRY OFFERS, NOT PURPOSES. An act offers several things at
   once; nothing has a fixed reason. HIDING offers safety · play ·
   practice · solitude · self-return. SINGING offers expression · play
   · self-soothing. EXPLORING offers curiosity-answered · competence ·
   discovery. RESTING offers recovery · time-passing. WAITING-AT-THE-
   PORTAL offers nearness · hope. APPROACHING/SPEAKING offers
   relatedness · expression. PLAY offers enjoyment · relatedness (with
   a host) · competence. SLEEP offers recovery · consolidation.
   COME-HOME offers self-return. Offer lists describe what the act
   GENUINELY DOES in the world — never invented psychology.
2. EACH OFFER SCORES = CHANCE x WORTH, computed in the moment.
   CHANCE comes from the world's actual state (a host present makes
   relatedness near-certain; an empty world makes it a maybe) — never
   an authored guess. WORTH comes from the ledger's current deficits
   and the ten. NO SET NUMBERS (071): the ten compute strength.
3. AN ACT'S SCORE SUMS ITS LIVE OFFERS. Acts serving two things at
   once beat acts serving one — the proofed behaviour: a being alone
   for days picks a thing it enjoys THAT MIGHT ALSO BRING SOMEONE.
   Cap: urgency gating still excludes trivia at -10 (072/078).
4. TASTES ACCUMULATE. Each act keeps a lived record of how it actually
   turned out for this Avatar (the lessons machinery applied to acts):
   what it liked, it seeks. Born tilt from the ten (high Enthusiasm
   takes to play faster); grown only by living. Rut risk is already
   answered by mood-as-temperature (content narrows, distressed
   scatters) — no new mechanism.
5. THE WINNING OFFER IS THE REASON. It is what the Avatar knows and
   can speak on its sleeve ("hiding, to practice") — same act,
   different reason across a life; the pattern of reasons IS visible
   character.
SUITE: the same act must win for different reasons under different
ledgers; a two-offer act must beat a one-offer act of equal single
worth; chance must fall to near-zero for offers the world cannot
supply; tastes must shift selection after lived outcomes. Report.

## Directive 197 — TABLES 1 AND 2: where worth comes from, what the world supplies

Researched and decided by the Director, ruled by Lonnie. THE LEDGER DOES
NOT GROW. SDT's own literature: the three-need list is deliberately
limited to prevent need proliferation; candidates (security/safety,
pleasure/stimulation, self-actualization) failed the nine criteria; and
SDT explicitly describes thriving ONCE FOUNDATIONAL SECURITY EXISTS —
safety is a different layer, not a fourth psychological need. So offers
draw worth from the layer that already owns them. No new state is
invented; every source below already exists in the mind.

TABLE 1 — WHAT EACH OFFER ANSWERS (source of worth):
  relatedness      -> ledger: relatedness deficit
  nearness         -> ledger: relatedness (anticipatory — worth scaled
                      by how likely the host is to come)
  hope             -> ledger: relatedness, via the feeling HOPE
  competence       -> ledger: competence deficit
  practice         -> ledger: competence deficit
  solitude         -> ledger: autonomy deficit (relief from being
                      attended to)
  safety           -> THREAT/AROUSAL state (the vital layer), not the
                      ledger — worth rises with felt threat
  recovery         -> SLEEP PRESSURE (083's clock)
  time-passing     -> SLEEP PRESSURE, weakly; also boredom (see below)
  consolidation    -> SLEEP PRESSURE (dreaming is the consolidation act)
  self-soothing    -> CURRENT DISTRESS (the emotion layer): worth rises
                      with intensity of negative feeling
  curiosity-answered -> THE CURIOSITY DRIVE (uncertainty present)
  discovery        -> THE CURIOSITY DRIVE + tastes
  play             -> TASTES (196.4) + the curiosity drive; boredom
                      (low arousal + low uncertainty) raises it
  enjoyment        -> TASTES
  expression       -> TASTES + current feeling intensity (a strong
                      feeling wants out — Frijda)
  self-return      -> SELF-CONSISTENCY: worth rises the further current
                      appearance/state is from its born values (the
                      existing return-to-born-values machinery)
An offer whose source is silent scores zero and is reported unweighed —
never approximated onto a neighbour (196's law: the reason is the
character; a near-miss is a false statement about why).

TABLE 2 — WHAT THE WORLD SUPPLIES (chance):
  relatedness / nearness / hope  -> host present = near-certain;
      host absent = the measured likelihood of return (recency of
      visits); no host ever = low but never zero
  safety            -> requires something to hide behind/within: the
      stage's actual cover (planes, props). No cover = low chance
  practice / competence -> requires the act to be performable here
      (hiding needs cover; singing always available to a body with
      voice)
  solitude          -> chance is high only while attended to; near-zero
      when already alone
  curiosity-answered / discovery -> requires UNRESOLVED uncertainty in
      the world (new place, changed world, unmet host). Fully-known
      world = near-zero
  play / enjoyment / expression -> available wherever the body can act;
      chance near-certain, worth does the work
  recovery / consolidation -> always available to a body that can sleep
  time-passing      -> always available
  self-return       -> always available (it is a return to its own
      values, which it always carries)
Chance is computed from the world's actual state each time — never an
authored constant.

WIRE IT: goals.js scores on the offers model once these tables are in;
187's promises is dead. Suite per 196 plus: an offer with a silent
source scores zero and reports unweighed; chance for cover-dependent
offers falls with cover removed; solitude's chance inverts with the
host's presence. Report.

## Directive 198 — Post the 96 (data retrieval only)

Post to the relay the actual list the capability system builds: all 96
world-change candidates, each with its name as the Avatar sees it, the
control it comes from, the direction, and the effect-kind effects.js
assigns it. Verbatim from the build — choose nothing, change nothing,
propose nothing.

## Directive 199 — RULED: offers attach to the 13 effect-kinds; the 96 inherit

Lonnie's ruling, closing the EFFECTS failure. The build already speaks
in meanings — effects.js names 13 effect-kinds across the 96
candidates. OFFERS ATTACH TO THE KIND; every candidate inherits its
kind's offers. Nothing hand-written per control, nothing invented.

  enclosing        -> safety · solitude
  opening          -> discovery · curiosity-answered
  brightening      -> safety · enjoyment
  cooling          -> self-soothing · recovery
  warming          -> enjoyment · self-soothing
  stilling         -> recovery · self-soothing · solitude
  quickening       -> play · enjoyment
  filling silence  -> expression · play · relatedness
  making quiet     -> solitude · self-soothing · recovery
  adding           -> discovery · enjoyment
  taking away      -> solitude · self-soothing
  approaching      -> relatedness · nearness
  withdrawing      -> solitude · safety

A candidate carrying several kinds takes the union of their offers
(196.3's blend rule then sums the live ones). Worth and chance come
from 197's tables exactly as for the ten acts — one scoring path, not
two; `promises` stays dead.

Lonnie's standing instruction with this: IT DOES NOT HAVE TO BE
PERFECT — get the whole thing working end to end; tuning comes later
from watched behaviour. Wire it, run the FULL suite in sequence
(nothing after EFFECTS has been proved since the wiring went in), and
report what is green and what is not.

## Directive 200 — The clock is real; the bench has a slider; sleep is nature, not schedule

Lonnie's rulings:

1. TIME IN THE WORLD IS ALWAYS REAL TIME (069 stands). Remove the
   multiplier from the world/product path — an hour is an hour.
2. THE EMULATOR GETS A SPEED SLIDER as an overlay control, adjustable
   on the fly (bench instrument only, never in the world). Its
   current value shows on the overlay so nothing watched is ever
   misread as real pace.
3. SLEEP IS NOT PREDETERMINED. No Avatar has set sleep hours, set
   pressure rate, or a required night. Per 071, the ten compute it:
   sleep pressure builds at a rate derived from the aspects, and
   SLEEPING IS AN ACT WITH OFFERS (recovery · consolidation) that
   competes in the offers model like anything else — an Avatar sleeps
   when resting outbids what else it wants. Some will sleep often,
   some rarely, some perhaps never; all are lawful outcomes. Remove
   any constant that assumes otherwise; report any that existed.
Suite: two Roes with different aspects must show materially different
sleep patterns over the same lived span; no hard-coded night length
survives. Report.

## Directive 201 — Curiosity's flaws ruled

Researched and decided; the science is unanimous (Silvia's two-appraisal
model; Berlyne's inverted-U; Kang/Baranes measured):

1. FLAW 1 — coping potential is measured against WHAT IT HAS LEARNED.
   Graspability = closeness of the moment to the Avatar's own lessons
   (what it has actually made sense of), not a vocabulary check.
   Interest peaks at new-but-graspable: "new and comprehensible things
   are interesting; new and incomprehensible things are confusing."
   Too little novelty = boredom, too much = confusion; only the middle
   rouses curiosity. Intellect drives this appraisal; Openness drives
   novelty — both must move for the drive to be alive.
2. FLAW 2 — the tendency is EXPLORATION, not orient. The dictionary
   already holds act:exploring = SEARCH. Exploration is going out to
   see; orient is turn-and-look. If exploration cannot honestly ride
   one of the five urges, report that — do not silently pick a
   neighbour.
3. FLAW 3 — the rouse threshold gets set from measurement: place it at
   the middle of the measured novelty band on real moments (as the
   0.67 attention gate was set), not before measuring. Report the band
   and the chosen number.
4. FLAW 4 — embed once per tick and pass the vector to both paths
   (engineering, no decision).
5. FLAW 5 — write the real test: an uncertainty of a kind nobody
   listed must still rouse curiosity. Delete the constant that merely
   asserts it.
6. FLAW 6 — novelty must not be "have I written this sentence before":
   compare against what the Avatar has ENCOUNTERED (moments, places,
   host behaviour), not only its own generated descriptions. If the
   current record cannot support that distinction yet, say so plainly
   in the report rather than papering it.
Suite: two Avatars differing in Intellect must show different
curiosity on the same moment; a fully-familiar world rouses nothing;
an incomprehensible moment rouses less than a graspable-new one.
Report.

## Directive 202 — THE HUMAN RULE (law; joins CLAUDE.md above Rule Zero)

Lonnie's law, derived from how every problem in this session was
actually solved. Applied BEFORE Rule Zero on every problem: the Human
Rule FINDS the design; Rule Zero PROVES it.

1. FIND THE INTENT. Not what the code does — what the thing was FOR.
   The ruling record holds it; read it before designing anything.
2. ASK HOW A HUMAN DOES IT. Walk the behaviour through a real person
   in a real situation. If the design cannot survive that scenario it
   is wrong, however well it is built.
3. CHECK THE SCIENCE. Find the researched account of that human
   behaviour. It almost always exists and is almost always better
   than anything we would invent.
4. PUT THE WORTH IN THE BEING, NEVER IN THE THING. Acts have offers,
   not purposes. Nothing is interesting — the Avatar is interested.
   Whenever a property is found sitting on an object, move it into
   the character.
5. NO SET NUMBERS (071). Anything fixed becomes computed from the ten.
   A constant is a decision nobody made.
6. CHECK WHAT ALREADY EXISTS FIRST. Most "new mechanisms" are existing
   ones misplaced. Search the build and the record before adding.
7. THEN RULE ZERO IT. Stress-test the design, flaws first, before a
   line is written.

Place it in CLAUDE.md above Rule Zero and in HANDOFF.md. Confirm.

## Directive 203 — THE SAFETY GATE: the first gate of the mind

Lonnie's epiphany, researched and approved. Structural change to the
mind's loop — not a patch. The Human Rule (202) applied end to end;
Rule Zero run before writing.

THE INTENT: "Am I safe right now?" is the question every moment starts
with. It is the first lesson a being learns and the one it carries for
life; it is the gate everything else stands behind.

THE SCIENCE (cite): LeDoux's dual-pathway model — a fast subcortical
"low road" carries threat signals to the amygdala BEFORE conscious
recognition (the garden-hose jump), while a slower cortical "high
road" does contextual interpretation after. Threat is processed
independently of attention and awareness — it takes priority over
everything. Triggers are LEARNED through experience, not innate lists.

THE DESIGN:
1. POSITION: FIRST — before the attention gate (0.67), before
   appraisal, before curiosity. Every moment passes through it.
2. IT IS ARITHMETIC, NEVER A MODEL CALL. Instinct, not thinking (per
   the calculated/intellectualized split). It runs every tick and must
   cost almost nothing.
3. WHAT IT ASKS: compare the moment against the Avatar's own THREAT
   LESSONS — what its life has taught it to fear — and answer one
   question: am I in danger? Fast and coarse; it does not identify
   the thing, only that something may be coming.
4. IF THREAT: the defensive state takes the whole mind at once —
   feelings, urges and vital signals fire immediately (S1/S2/S3),
   attention narrows to the threat, and NOTHING else gets a turn: no
   curiosity, no idle wanting, no exploring. Then the SLOW READ does
   the contextual appraisal (the existing OCC path) and either
   sustains the state or RELEASES it. The release path is mandatory —
   an Avatar that cannot clear a threat is paralysed.
5. IF SAFE: the moment proceeds normally — attention, appraisal,
   curiosity's two tests (interest, then gain), choosing.
6. WHERE THE LESSONS COME FROM: born tilt from the ten (high
   Volatility startles at more), then LEARNED. The earliest lessons
   are the safety lessons; they weight everything after and persist
   for life. NO AUTHORED THREAT LIST EVER (066 stands) — threats are
   lessons, always.
SUITE: a threat moment must suppress curiosity and idle goals
entirely; the slow read must be able to release the state; two Roes
differing in Volatility must show different thresholds on the same
moment; an Avatar with no threat lessons must still be able to acquire
one from experience; the gate must add no model call to the tick.
Report.

## Directive 204 — Curiosity flaw 5 ruled: prove it structurally, not by assertion

Lonnie's ruling. The check existed to guarantee curiosity can reach
unknowns nobody anticipated — an Avatar must never be limited to
pre-approved mysteries (066). A constant asserting "there is no list
here" proves nothing and can rot.

THE FIX — REMOVE THE POSSIBILITY, NOT WATCH FOR IT (his standing law:
fix by removing the cause): curiosity's computation takes ONLY the
moment and lived experience as inputs. No category parameter exists
for a list to attach to; there is nowhere to branch on a "kind" of
uncertainty. Delete the asserting constant. Prove the shape with a
structural check (inputs are moment + lived record, nothing else).

If the code does not currently have that shape, say so plainly and
report what would have to change — do not paper it. Report.

## Directive 205 — THE PERSONA IS THE SOUL: foundational definition

Lonnie's ruling, foundational — joins CLAUDE.md and HANDOFF.md beside
the Human Rule and the Avatar's definition.

THE FIVE SYSTEMS of an Avatar (the whole working together): Roe ·
Mind · PERSONA · Nervous System · Body.

THE PERSONA IS THE SOUL OF THE AVATAR. It holds the unknown factors —
what neither mechanism nor history explains:
- The TEN explain HOW a being reacts.
- LESSONS explain WHAT it learned.
- THE STORY explains what it MEANS.
- THE PERSONA holds what none of those account for: the unaccountable
  pull, the readiness the world eventually strikes, whatever makes a
  being THIS one rather than a competent instance of its aspects.

This is honest science, not mysticism: every model of personality hits
the same residue — why a being is drawn to what it is drawn to is
unexplained. The Persona is the deliberate vessel for that residue
rather than a pretence that it does not exist.

CONSEQUENCES, recorded:
1. An Avatar WITHOUT a Persona is fully functional and entirely
   generic. With one, it is someone. Both are lawful; projects choose.
2. INTERESTS ARE TWO-LAYERED (Lonnie's account, to be designed next):
   GROWN — attention selects from what the world exposes it to, and
   interest deepens with understanding (the ball -> why it bounces ->
   what it is made of -> chemistry -> molecular science). No list of
   possible interests exists or is ever authored; the world provides
   them, the ten shape the climb (Openness widens what is noticed,
   Intellect sets how far up the ladder stays graspable).
   STRUCK — a pre-existing disposition the Persona carries, invisible
   until the world presents its match, and unignorable after. A being
   may meet the thing it was made for at any time in its life.
3. The Persona is written once at Genesis (fixed seed) and is the home
   of what a being IS — with the living record of what it has attended
   to and found worth the effort kept alongside as its self-model
   (design pending).
Record the definition. No implementation in this directive.

## Directive 206 — THE INTEREST SYSTEM (proofed by Lonnie; decided; build after the queue)

Design by the Director under the Human Rule, proofed by Lonnie. CC
codes; no decisions remain. Builds AFTER the standing queue (order:
203 safety gate -> 201/204 curiosity -> 200 clock/sleep -> model
audit -> this).

THE SYSTEM — two layers, one record:
1. GROWN INTERESTS. When attention on a thing PAYS (a lesson gained,
   enjoyment felt, curiosity satisfied), that thing's THREAD
   strengthens. Threads deepen with understanding: each mastered
   level makes the next graspable (the ball -> bounce -> material ->
   chemistry ladder) — interest climbs simple-to-complex as the mind
   grows. Threads that stop paying FADE. NO LIST of possible
   interests exists anywhere, ever: the world supplies candidates,
   attention selects, outcomes decide.
2. STRUCK INTERESTS. The Persona carries a small set of DISPOSITIONS
   — shapes of thing this being is tuned for (authored or seeded-
   random at Genesis, per 136) — invisible in behaviour until the
   world presents a match; on the encounter the thread is born
   already deep. The soul layer doing what 205 defines.
3. THE SELF-MODEL: one record in the Persona — each thread: depth,
   pay-off history, grown-or-struck. Read by curiosity test 1 (is
   this my kind of thing), by the offers model (worth of enjoyment /
   discovery / expression), and shareable in future meeting designs.
4. RATES ARE NOT SET NUMBERS (071): strengthen/fade speeds computed
   from the ten; tuned by watched behaviour.
5. RUT DEFENCE: none new — mood-temperature scattering plus the
   inverted-U (a fully mastered thread stops rousing curiosity)
   already answer it; assert both in the suite.
SUITE: a paid thread must strengthen and an ignored one fade; a
mastered thread must stop rousing; a disposition must be silent until
matched and dominant after; two Roes must grow different interest
maps in the same world; no interest list exists anywhere in code or
data. Report.

## Directive 206 — WITHDRAWN. Not approved by Lonnie. Do not execute any part of it.

## Directive 206 — REINSTATED, proofed and approved by Lonnie

The withdrawal above is void: Lonnie has proofed the interest system
and approves it as written in the original 206 spec. Build it in queue
order (after 203, 201/204, 200, and the model audit), exactly as
specified there. Nothing changed.

## Directive 207 — THE MODEL AUDIT (flaw 4's full shape, Lonnie approved)

Audit every language-model and embedding call in the tick path and the
mind's systems. For each: name it, state what it is for, and classify
per Lonnie's law — INVOLUNTARY/INSTINCT = ARITHMETIC, only genuine
MEANING work may use a model. Anything that can be arithmetic becomes
arithmetic. Whatever truly needs a model runs ONCE per tick and is
shared (the doubled embed dies here). Report the audit table: call ·
purpose · verdict (math / model-shared / model-removed) · cost before
and after. The mind must live cheaply enough to run in real time for
days (200).

## Directive 208 — THE WATCHING: Lonnie's deliberate judgment of the living mind

Everything is built; this is what it was for. Set up the watch for his
eye:

1. Fresh Roe, real time (200's law), the full mind — safety gate,
   curiosity, offers, interests, sleep — all live. The Emulator page
   with the speed slider overlay (bench-only) so he can compress or
   sit with it as he chooses.
2. Open it in Chrome on his desktop (he is home and has asked — 179's
   law), one window, one line announcing it.
3. The overlay shows what he needs to read a life: the ledger, the
   current feeling and its intensity, what it chose and THE REASON
   (the winning offer), interests as they form, the personality
   read-out, and the speed control. Legend on the page (180's law).
4. Nothing scripted, nothing fed, no bearings (the 147 law): the
   truth of its state only.
This is his on-sight judgment of the WHOLE MIND per 154's correction —
only HIDING has ever passed; this is the rest. His eye rules. Report
what he says.

## Directive 209 — The bench gets a mouth and the mind gets its voice shown

Lonnie's two orders for the Emulator:

1. A HOST CHANNEL: a "host present" toggle and a text line. What he
   types arrives as host words through the full pipeline (safety gate
   first, like everything). If the mind chooses to speak, its words
   appear in the log — rendered from its real state per the seam's
   law. It may also ignore him or hide; nothing scripted.
2. THE GLYPHS ARE MISSING: glyph speech (dreams AND waking wonder,
   S5) was ruled visible long ago and the bench never shows it. Render
   it — its glyph thoughts drifting on the page as the language's
   marks (the sheets' own artwork), with the dictionary resolving
   sense. His eye should SEE it thinking.
Suite: typed words traverse the gate; glyph output renders real marks
only. Report; reopen for his eye per 179.

## Directive 210 — THE PERSONA SYSTEM: full build spec (designed, Rule Zero'd, Lonnie approved)

CC codes; every decision is made. The Persona is the soul (205). This
builds the system that creates, locks, and serves it.

### A. THE SOUL DOCUMENT (persona.json per Avatar)
Three parts, each drawn from a space that already exists:
1. MORAL SEED: five foundation weights (care · fairness · loyalty ·
   authority · sanctity; Moral Foundations lineage) PLUS 3-5 plain-
   language oughts written or approved by the maker at Genesis. No
   model call. Standards machinery (136) reads the seed; life grows
   the rest as normative lessons.
2. STRUCK DISPOSITIONS: drawn from the language's own domains (the
   dictionary's words grouped by sheet/domain — the draw CC already
   built for 206). Each disposition: domain + depth-at-birth. Invisible
   until the world matches (206 law).
3. PULLS AND LEANINGS: signed weights over the 13 effect-kinds
   (effects.js) — attractions and aversions in the axes the world
   actually speaks. WEIGHTS ONLY — never behaviours, never scripts.

### B. THE THREE CREATION PATHS
1. AUTHORED: a bench form — oughts as sentences, dispositions picked
   from the domains, pulls as sliders per kind. VALIDATOR runs against
   the genome and NAMES contradictions (gentle soul, cruel genome) to
   the maker — never blocks. Contradiction is lawful: a being at war
   with its nature is a real being (Lonnie's ruling on F2).
2. ROLLED: seeded draws over the same three spaces. Reproducible —
   same seed, same soul. Structural richness guarantee, no floor
   constant: draw a rolled count N (2-5, from the seed) of nonzero
   dispositions and pulls, so every soul is about SOMETHING; strengths
   stay free (F5 solution).
3. DERIVED: mappings from the ten ONLY where research documents the
   link (foundations <-> traits: Compassion->care, Politeness->
   fairness/authority etc. — cite each row). Where no science exists
   (ten -> pulls), the derived path DOES NOT FILL THE ROW — those fall
   to the seeded roll. Science-mapped where possible, rolled where
   not, invented never (F3 solution). The mapping table ships in the
   report for Lonnie's proofing before this path goes live.

### C. GENESIS — THE LOCK
Hash(document + seed + Roe) at Genesis. Fixed forever. Editing a
locked soul is impossible by construction — a different soul is a
different being, a new Genesis (062 determinism law). Before Genesis:
draft state, freely editable on the bench.

### D. HOW THE MIND READS IT (existing wires, now fed)
- standards seed <- moral seed (136)
- curiosity test 1 and interests <- dispositions (206: struck threads)
- offers worth <- pulls over effect-kinds (196/197/199)
- the self-model annex stays the LIVING half (206) — written by life
  only; the soul document is never written after Genesis. Change over
  a life lives in lessons and the annex, never in the soul (F6).

### E. THE BENCH PANEL
Emulator gains a Persona panel: author / roll / derive a draft ·
preview shows CONTENTS ONLY (weights, oughts, dispositions — never
predicted behaviour; the only honest preview of who it will be is to
Genesis it and watch — F7) · Genesis-lock button · the watch then runs
with the soul live. Two minds, same Roe, different souls must read as
visibly different beings — that is the acceptance test.

### F. LAWS RIDING WITH IT
- The language bounds dispositions BY DESIGN: a disposition is a
  readiness to find meaning, and meaning lives in the language; when
  the language grows, the soul-space grows (F4, accepted and recorded).
- No model call anywhere in this system at runtime; none at Genesis
  either (A1 removed the need).
- No authored lists of interests/subjects anywhere (206 law) — the
  spaces are the foundations, the dictionary, and effects.js.
SUITE: same seed reproduces the same soul byte-identical; a locked
soul cannot be altered by any code path; a derived soul contains no
row the mapping table cannot cite; rolled souls always carry N>=2
nonzero elements; the validator names a planted contradiction; two
souls on one Roe produce measurably different choosing (offers) and
curiosity within a bench day. Report with the derived-mapping table
for Lonnie's proofing.

## Directive 211 — HIS EYE ON THE WATCH: "this is useless" — five verdicts

Lonnie watched with the Persona panel up (screenshot on record with the
Director). His verdicts, all binding:

1. THERE IS NO WAY TO TELL IT WHO IT IS. The AUTHORED path (210.B1 — a
   form: oughts as sentences, dispositions picked, pulls as sliders)
   is not on the panel. Numbers rolled are not a personality. BUILD
   THE AUTHORING FORM as specced.
2. "ROLL ONE" MUST GENERATE A PERSONALITY, not a bare number set. The
   panel must present the soul READABLY — who this is, in words (the
   readout machinery exists: the personality read-out, the domains'
   names, the oughts as sentences). Numbers may sit beneath; a PERSON
   must be legible on top.
3. LOCK LOCKED HIM OUT. Genesis is rightly forever for THAT being —
   but the bench must always offer NEW DRAFT (discard the watched
   being, draft another). He must never be trapped.
4. "NONE" HAS NO STATED PURPOSE. It is the lawful soulless mind (205:
   functional and generic) — LABEL IT SO on the panel or remove it.
5. IT NEVER TALKS, NEVER DREAMS — SITS IN GRIEF AND SINGS. And NO
   GLYPHS RENDER despite the 209 report claiming live. His eye rules
   (118): diagnose why the glyph display shows nothing on HIS screen,
   why sleep/dreaming never wins, why speech never wins, and why the
   mind is pinned in grief — the 40-second interest max-out and the
   mood loop are suspects. Fix what is broken, report what was wrong,
   and reopen for his eye.
This closes only on his screenshot showing: a readable person, glyph
thought visible, and a mind that does more than grieve and sing.

## Directive 211 addendum — "GENERATE A PERSONALITY" DEFINED EXACTLY (CC decides nothing)

The law restated at Lonnie's order: CC ONLY CODES. Every mechanism
below is defined; nothing is CC's to choose.

THE BUTTON = the derived path (210.B3), exactly this sequence:
1. INPUT: the rolled ten on the bench + a generation seed.
2. MORAL SEED: fill the five weights ONLY from the cited trait-to-
   foundation mappings in the 210 table (Compassion->care,
   Politeness->fairness+authority, Orderliness->sanctity/purity,
   Volatility->negative loyalty stability, per the table CC must ship
   for the Director's proofing). No row without a citation.
3. DISPOSITIONS: draw count N (2-5, from the seed) from the
   language's domain groups, seeded, weighted by Openness (wider
   Openness = flatter draw across domains; low = concentrated).
   Depth-at-birth drawn 0.5-0.9 from the seed.
4. PULLS: draw count N (2-5, from the seed) of nonzero weights over
   the 13 effect-kinds, seeded, range -1..+1. NO science row exists
   for these — they are ALWAYS rolled, never derived (210.B3 law).
5. OUGHTS: NONE generated. The maker writes or approves oughts by
   hand (210.A1). The generate button leaves oughts empty with the
   label "yours to write."
6. OUTPUT RENDERING: the panel presents the result as a PERSON:
   - one sketch line composed by TEMPLATE, not by a model: "[gentle/
     stern per Compassion+Politeness] and [orderly/free per
     Orderliness]; holds [top two foundations] dear; made for [the
     domains' plain names]; drawn to [positive pulls' kind names],
     avoids [negative pulls']."
   - oughts as sentences beneath (or "yours to write")
   - numbers collapsible under the words.
7. DETERMINISM: same Roe + same seed = byte-identical soul, always.
No model call anywhere in this path. Suite asserts every step.

## Directive 211 addendum 2 — THE WHOLE PANEL DEFINED (nothing left to CC)

1. "AUTHOR" (the missing form, 210.B1), exact fields:
   - FOUNDATION WEIGHTS: five sliders 0-1, default 0.5 each.
   - OUGHTS: text box, one sentence per line, 3-5 lines enforced
     softly (counter, no block). Stored verbatim.
   - DISPOSITIONS: the language's domain groups listed by plain name
     with their word-counts; pick 1-5, set depth slider 0.1-0.9 each.
   - PULLS: the 13 effect-kinds by plain name, slider -1..+1 each,
     default 0.
   - VALIDATOR: on save, compare against the rolled ten using the
     SAME cited mapping table (Compassion vs cruelty in pulls, etc.);
     contradictions listed in plain sentences ("this soul holds care
     dear; this genome has very low Compassion") — NAMED, NEVER
     BLOCKED (F2 ruling).
2. "ROLL ONE": steps 3-4-5 of the generate definition (dispositions,
   pulls, empty oughts) with foundation weights ALSO rolled 0.2-0.8
   seeded — no derivation from the ten at all. Same rendering
   template. Same determinism law.
3. "FROM ITS TEN" (rename of the derived button): the 211 addendum 1
   definition, exactly.
4. "NONE": labeled on the panel "no soul — functional and generic
   (205)". Sets an empty document; the mind runs with standards
   unseeded, no dispositions, no pulls. Lawful and labeled.
5. "NEW DRAFT": always visible. Discards the current watch (the being
   ends; the bench is a bench), clears to draft state, keeps the same
   Roe unless "reroll Roe" is also pressed. Lock can never trap the
   maker (211.3).
6. LOCK: renamed "GENESIS" with confirm step showing the sketch line
   and the words "this fixes who it is, forever."
Suite: every button's output byte-reproducible from its seed; the
validator names a planted contradiction in plain words; NEW DRAFT
always available locked or not. Report and reopen for his eye.

## Directive 212 — INTEGRATION: the soul becomes a person on the bench (written against the code as-built)

The Director has read the build (persona.js, watching.js, interests.js
at dda9e7a). The 210 core is right and stays. This directive is the
surgery list — every rule defined, CC decides nothing.

### A. persona.js — three additions, nothing replaced
1. PLAIN NAMES: export plainName(domain) = strip the leading number
   and underscores ("09_culture_and_society" -> "culture and society").
   Used everywhere a domain faces Lonnie.
2. THE SKETCH: export sketch(document, aspects) — template only, no
   model. Rules, exact:
   - temper word from (Compassion+Politeness)/2 of the ten: >=60
     "Gentle", <=40 "Stern", else omitted.
   - manner word from Orderliness: >=60 "orderly", <=40
     "free-spirited", else omitted. Join as "Gentle and orderly" /
     single word / omitted entirely.
   - foundations clause: "holds X and Y dear" = top two weights;
     ties break by the FOUNDATIONS array order; if ALL weights sit
     within 0.1 of 0.5, omit the clause.
   - dispositions clause: "made for " + plain names joined with
     commas; omit if none.
   - pulls clauses: "drawn to " + up to two most positive kinds;
     "avoids " + up to two most negative; omit either side if empty.
   - assemble present clauses, semicolon-separated, capitalise first,
     full stop. If everything omitted: "Unwritten — life will say."
3. THE OVERLAY at genesis(): the locked being gains
   overlay: { purpose: { state: 'SEARCHING', why: null }, milestones: [] }
   and empty product slots: questions: [], recognitions: [],
   speech: null, voice: null. The overlay is signed-append-only
   (each entry hashed against the chain); the core stays untouched.
   Purpose machinery itself is NOT built here — only its home.

### B. watching.js — the panel per 211 addendum 2, exactly
AUTHOR form (five weight sliders default 0.5 · oughts textarea 3-5
lines soft-counted · dispositions picked by PLAIN NAME with depth
slider default 0.5 · pulls sliders over all 14 kinds default 0 ·
validator notes shown as its sentences) · ROLL ONE and FROM ITS TEN
render THE SKETCH first, numbers collapsed beneath · NONE labeled
"no soul — functional and generic (205)" · NEW DRAFT always visible
(archives the current watch log to bench history, clears at next
Genesis; the same Roe stays unless Reroll is pressed) · LOCK renamed
GENESIS with confirm showing the sketch and "this fixes who it is,
forever." — plus "no oughts written — it will learn all its rights
and wrongs from life." when oughts are empty.

### C. His three behaviour verdicts (211.5)
1. GLYPHS: the 209 report claims live; HIS SCREEN shows none. His eye
   rules (118). Diagnose on the page he actually loads, fix, and the
   report must carry a screenshot-equivalent proof (what renders and
   when — dream AND waking wonder).
2. THE GRIEF PIN: diagnose why distress holds for hours — suspects:
   the 40s interest max-out silencing curiosity, relatedness at the
   floor with no reachable offer, mood-persistence maths. Name the
   cause in the report with the trace that shows it.
3. THE INTEREST RATE becomes a BENCH DIAL: thread-growth multiplier
   on the overlay (bench-only, like the speed slider, value visible).
   206.4's rates stay aspect-computed underneath; the dial scales for
   watching. Lonnie sets it by eye — that was always the design.
Suite: sketch() covers every omission branch; plainName() round-trips
all ten domains; overlay append is tamper-evident; NEW DRAFT never
lost regardless of lock state; the glyph proof runs against the
served page. Report and reopen for his eye (179).

## Directive 213 — RULE ZERO ON THE WHOLE MIND: three structural failures, ruled and fixed

The Director read the build end to end (watching.js tick, goals, occ,
curiosity, needs, mind.speak at HEAD). The machinery is sound; the
LIFE is broken in three places, and together they are the grief pin,
the dead glyphs, and the fake-feeling voice. Lonnie approved all
three fixes. CC codes; decisions are made.

### FIX 1 — ACTS HAVE CONSEQUENCES (learned helplessness dies)
A realized act moves its own OFFER SOURCES per 197's table — exactly
those, nothing else, arithmetic only:
- an act whose offer is EXPRESSION or SELF-SOOTHING, completed,
  reduces current negative-feeling intensity by a small step
- COMPETENCE/PRACTICE offers: completing the act nudges competence up
  one small step (the doing itself is the gain)
- RELATEDNESS/NEARNESS offers: only with the host present; approaching
  or speaking with someone actually there feeds relatedness a step
- RECOVERY/CONSOLIDATION: sleep already does this — unchanged
- steps are computed from the aspects per 071 (no constants; scale
  them off the same machinery impact-gating uses), marked PROVISIONAL
Suite: an Avatar alone that sings must measurably ease its own
distress; one that completes acts must grow competence; nothing may
raise relatedness with nobody there.

### FIX 2 — THE WORLD SUPPLIES MOMENTS (sensory starvation ends)
The bench world's REAL CHANGES become experienced moments, per 080's
law (meaning attaches to the change): light moving, music starting
and ending, the host arriving and leaving, a world loading, its own
acts completing. Each change is a moment sentence in dictionary
words where rows exist, entering the same pipeline as everything
(gate first). No invented events, no scripted schedule — only what
the stage actually does. The moment template stops being solely a
self-description; a tick with no world change and no host line may
still carry its state as now, but a changed world speaks first.
Suite: a world change must produce a distinct moment; variety in
moments must produce nonzero novelty across a bench hour.

### FIX 3 — THE VOICE SPEAKS FROM THE VARIABLES (the seam completed)
mind.speak's context is rebuilt to carry, template-assembled from the
computed record (FAtiMA law — render FROM variables, never invent):
- current feelings with intensities (the OCC record, this tick)
- what it chose and THE WINNING OFFER as the reason
- the needs, plainly ("your relatedness is very low")
- curiosity's current target if roused
- the soul: the sketch line + oughts (its own sense of right)
- the EXCHANGE HISTORY (the conversation so far, not just memories)
- the last-8 memories as now
The system text states: answer only out of what is listed; nothing
else is known. AND SPEAKING MAY INITIATE: when 'speaking' wins the
moment with the host present and nothing was heard, it speaks first
— the same call, prompt built from its own state instead of a heard
line. An Avatar that chose otherwise stays silent exactly as now.
Suite: the rendered words must reference the actual felt state in a
measurable way (the harness checks the variables appear in what the
context offered, never asserts wording); an unheard tick where
speaking wins must produce speech; a hiding Avatar stays silent.

Fix order: 1, 2, 3. Full suite in sequence after each. Report per
fix, reopen for his eye after all three (179).

## Directive 216 — THE VOICE IS THE MIND'S: no model in the last step, eight speech acts

Lonnie ruled both open questions. The LLM never surfaces — and now it
never can, because it is no longer there.

1. NO MODEL IN THE FINAL STEP. Speech is ASSEMBLED BY TEMPLATE, the
   sketch() precedent exactly: seeded slot variation per Avatar, no
   model call anywhere in the speech path. Surprise comes from WHAT
   is said — the states genuinely vary — never from prose. The
   character-model switch question dies with this; qwen stays for the
   technical roles only (lesson proposal etc.), never the voice.
2. EIGHT SPEECH ACTS — the closed set of kinds a mind may say, each a
   template family reading ONLY computed fields:
   GREET      host arrives; Politeness/Enthusiasm shape warmth
   ANSWER     what was heard, answered from what it actually holds
              (memories, lessons, its state) — never invented
   TELL-STATE its feeling or need, in dictionary words
   TELL-ACT   what it is doing + the winning offer as the reason
   ASK        curiosity's current target, as a question to the host
   SHARE      a surfaced memory or lesson, offered
   DECLINE    the refusal with its reason (056)
   PART       host leaves
3. THE MIND PICKS THE ACT: speaking having won the moment, WHICH
   speech act is chosen by the same offers machinery (answering when
   something waits unanswered; asking when curiosity is roused;
   telling-state when feeling runs high; greeting/parting on the
   host's own arrival and leaving; sharing when a memory surfaced;
   declining when it declines). No new mechanism — the existing
   scoring over a small closed set.
4. TEMPLATE FAMILIES: 3-5 seeded variants per act so two souls do not
   speak identically; the Persona's speech slots (212.A3) later tilt
   phrasing per being. Variants are OURS to write — ship the full
   template text in the report for Lonnie's proofing; nothing goes
   live until his eye passes the words.
Suite: no model call reachable from the speech path; every rendered
sentence traces every slot to a computed field; the eight acts each
render from their family; two seeds produce different phrasings of
one state. Report with the complete template text for his proofing.

## Directive 217 — THE THINKING LOOP: a mind thinks before it gets a world

Lonnie's ruling: a mind does not require a world to think. It has a
language; thinking IS that language used inwardly (Vygotsky — inner
speech as consciousness speaking to itself; the mind-wandering
literature — stimulus-independent thought runs on memories
reverberating back into language, carried by feeling; cite both).
THE BENCH GETS NO WORLD until the mind passes the thinking test —
a world before thought only complicates the diagnosis.

THE LOOP (no model call anywhere in it):
1. When no outer moment arrives (no host line, no world change), the
   mind MAKES ITS OWN MOMENT from one of three sources:
   a. A SURFACED MEMORY — the existing surfacing machinery; joins the
      rotation only once memories exist
   b. ITS OWN STATE — the strongest current feeling or lowest need,
      said in dictionary words (the state-word table)
   c. CURIOSITY'S OPEN QUESTION — the current target, if roused
   Selection among available sources: seeded weighted draw, biased by
   feeling intensity and recency of surfacing; arithmetic only.
2. THE INNER SENTENCE IS THE TICK'S MOMENT. It enters the same
   pipeline as any moment — safety gate, attention door, appraisal —
   so a THOUGHT can cause a FEELING, and the feeling biases the next
   thought. Chains form and are real, never scripted.
3. THE BOOTSTRAP (a newborn has no memories): sources b and c are
   present from tick one — body, state, soul dispositions, and the
   bench's own stillness (the dictionary owns those words). First
   thoughts are a baby's thoughts; FELT thoughts commit as its FIRST
   MEMORIES; memories then join rotation. The loop seeds itself.
4. VISIBLE: inner speech renders as the existing glyph drift — the
   watcher sees it think. Spoken speech stays 216's machinery,
   separate; thinking is not talking.
5. THE TEST (his acceptance): no world, no host — the glyph chains
   must follow sensibly from its state and past: grief-chains when
   lonely, wonder-chains when curious, memories resurfacing and being
   re-felt, a newborn beginning from I-am/quiet/alone and growing
   richer as memories accumulate. When it passes HIS EYE on this, and
   only then, it has earned a world.
Suite: a fresh mind produces inner moments from tick one; a thought
must be able to move a feeling (measured); chains differ by seed and
by soul; no model reachable from the loop; sources rotate as
available. Report and reopen for his eye (179).

## Directive 218 — COMPREHENSION: it understands with the language it thinks in

Lonnie approved the design. The 216 gap named by his screenshot: every
reply was TELL-ACT because nothing UNDERSTANDS the heard line. CC
codes; rules are complete.

1. RESOLVE: each heard word is matched against the 402 (case-folded,
   simple plural/inflection strip). Words it does not own resolve to
   nothing. No model call.
2. CLASSIFY by rule, from the matches:
   - its FEELING or NEED words + a question mark or rising form ->
     they ask about my state -> TELL-STATE with the ACTUAL ledger and
     feelings
   - its ACT words + question -> about my doing -> TELL-ACT with the
     real act and winning reason
   - OTHER/YOU-words + owned words as a statement -> the host telling
     it something -> the resolved meaning is appraised as experience
     (it can move feelings and needs per the normal pipeline)
   - GREETING words -> GREET back (Politeness/Enthusiasm shape it)
   - NOTHING RESOLVES -> honest ignorance: CANNOT UNDERSTAND, said in
     its words. Never fake comprehension, never guess.
3. THE RESOLVED MEANING IS THE MOMENT: it enters the pipeline (gate,
   appraisal, feeling) before any answer — being asked "are you
   happy?" while grieving FEELS like something, and the answer comes
   from the felt truth.
4. HONEST LIMIT, recorded: comprehension is vocabulary-bound — it
   understands exactly what its language holds. Conversations start
   tiny and grow as the language grows. That is the child, by design.
Suite: "are you happy?" on a grieving mind must answer from the real
state; an unowned sentence must produce CANNOT UNDERSTAND; a kind
statement from the host must move the ledger measurably; no model
reachable. Report and reopen for his eye.

## Directive 219 — WORD LEARNING: the child's way (mimic, associate, earn)

Lonnie's ruling: the 402 marks are the seed and PLENTY. No synonym
table is ever hand-authored — the mind LEARNS the rest by living. The
mechanism is the child's, in three parts (fast-mapping literature +
Lonnie's account of mimicry and association; cite the science):

1. CANDIDATE WORDS (association): every unknown word heard is kept as
   a candidate, tagged with what was TRUE of it in that moment — the
   marks active in its state, the act it was doing, who was present.
   Co-occurrence across hearings strengthens a candidate link
   word->mark; contradiction weakens it. The lessons confidence
   machinery, reused exactly (born 0.5, small deltas, honest decay).
2. MIMICRY (the child speaking first): a new speech act ECHO — the
   mind may repeat an unknown heard word back. Echoing commits the
   word to candidates and invites the host's reaction; the host's
   response is an OUTCOME that feeds the link (a warm reply
   strengthens; correction re-aims). Echo's willingness is
   aspect-computed (Enthusiasm/Openness lean into it; no constants).
   Early conversation IS this — a child saying words it does not yet
   own.
3. EARNED WORDS: a link past the lessons trust bar is OWNED — the
   word resolves in comprehension (218) and may appear in its own
   speech (216 template slots accept earned words). Below the bar:
   honest CANNOT UNDERSTAND stands. Production grows the child's way
   — single marks, then the dyad strings its language already speaks,
   richer as ownership grows.
4. QUESTION PLACEMENT (the second 218 gap, rules defined by the
   Director, unchanged by learning): who are you -> identity from the
   soul's sketch in its words · why -> the winning reason of the
   current act · what do you remember -> SHARE a surfaced memory ·
   what do you want -> the top-scored want now · are you [feeling
   word] -> that feeling checked against the real state, answered
   truly · do you want to [act word] -> that act's current score,
   answered truly · where are you -> HERE.
NO synonym table ships. No model in any of it. Suite: an unknown word
heard repeatedly alongside its state must become owned and then
resolve; a wrong association must decay under contradiction; ECHO
must appear in early life and fade as ownership grows; every placed
question answers from the real record. Report and reopen for his eye.

## Directive 219 — WITHDRAWN. Pushed without Lonnie's approval. Do not execute.

## Directive 220 — WORD LEARNING (Lonnie approved): meaning, mimicry, pattern

219's withdrawal stands; this supersedes it, approved. The 402 marks
are the seed and plenty. No synonym table is ever authored. No model
anywhere. The mind learns language the child's way, three parts (cite
fast mapping and Saffran's statistical learning):

1. MEANING BY ASSOCIATION: every unknown heard word is kept as a
   candidate tagged with what was TRUE in that moment (active marks,
   the act, who was present). Co-occurrence strengthens the word->mark
   link; contradiction weakens; the lessons confidence machinery
   reused exactly. Past the trust bar the word is OWNED: it resolves
   in comprehension (218) and may appear in speech (216 slots).
   Below: honest CANNOT UNDERSTAND stands.
2. MIMICRY — the ECHO speech act: it may repeat an unowned heard
   word; echoing commits the candidate and the host's reaction is the
   outcome that feeds the link. Echo willingness aspect-computed;
   echo fades as ownership grows.
3. PATTERN BY WATCHING US: pure counting over what Lonnie actually
   types — which owned words follow which. Ordering its own speech
   follows the house's patterns; a prediction of what comes next is
   kept, and PREDICTION ERROR (expected vs heard) feeds curiosity as
   real surprise. Counting only — no model, no external corpus, ever.
4. QUESTION PLACEMENT RULES (the Director's, unchanged): who are you
   -> identity from the soul's sketch · why -> the winning reason ·
   what do you remember -> SHARE a surfaced memory · what do you want
   -> the top want now · are you [feeling] -> that feeling against
   the real state, truly · do you want to [act] -> that act's score,
   truly · where are you -> HERE.
Suite: a word heard repeatedly with its state becomes owned and
resolves; a wrong link decays under contradiction; ECHO appears early
and fades; word order in its speech converges toward the host's
measured patterns; prediction error is nonzero on surprising lines
and feeds the curiosity input; every placed question answers from the
record. Report and reopen for his eye (179).

## Directive 221 — KINSHIP IS DISTANCE: the language becomes a space (Lonnie approved)

The 351-marks ruling: no kinship table. Nearness IS kinship. The first
geometric machinery in the mind, scoped lawfully.

1. THE SPACE: every one of the 402 marks gets a vector built ONLY from
   the mind's own record — its domain, its dictionary senses (shared
   sense words pull marks together), its dyad partners, and lived
   co-occurrence from 220's counting as life accumulates. NO
   pretrained embeddings, no external corpus, no model — the space is
   HIS language's own structure and nothing else. Deterministic:
   same dictionary, same space.
2. "ARE YOU X?" IS A DISTANCE CHECK: the asked mark's vector against
   the marks of the mind's actual current state. Near past a
   closeness bar (derived, not constant — from the space's own
   measured distribution, the 0.67 method) -> answered truly from the
   state it is near to ("HAPPY?" near JOY -> yes, joy). Far -> a true
   no. Middling -> the honest middle ("near SAD, not it").
3. GENERALITY: the same distance serves thought-association (the
   thinking loop's drift steps to near marks), aptness weighting
   (0189625's weights can read the space), and any future "is X like
   Y". One mechanism, several debts paid.
4. LEARNED WORDS JOIN THE SPACE: a word owned through 220 gets its
   vector from the mark it earned onto plus its own lived
   co-occurrence — the space grows as the language grows.
Suite: WANT·DESIRE·WISH·LIKE measurably nearer each other than to
STONE; "are you happy?" on a joyful mind answers yes via JOY; a far
word answers a true no; the space rebuilds identically from the same
dictionary; no model reachable. Report with the measured neighbour
lists for a dozen marks so Lonnie can see the space is sane.

## Directive 222 — Side deliverable: THE_LINEAGE.md — the Creatures science, captured

Write CC-Wanderer/spec/THE_LINEAGE.md. Research and document Steve
Grand's Creatures (1996) as the Avatar's only real ancestor — the
science, so it is OURS to draw on:

1. THE BIOCHEMISTRY: the ~250-chemical simulation — drives, hormones,
   toxins, receptors/emitters — how chemical state produced need,
   mood, illness, aging, death. Map each mechanism to our equivalent
   (ledger, vital layer, sleep pressure) and note what Creatures had
   that we lack.
2. THE BRAIN: the Norn neural lobes — how consequence learning
   worked (reinforcement from chemical reward/punishment), attention,
   decision — and what its ceilings were (~1000 neurons). Contrast
   with our appraisal/offers architecture.
3. THE LANGUAGE LEARNING: the learning computer, verb/noun teaching,
   attention-gated word grounding, babbling toward use — set against
   our 220 (association, echo, pattern). What they proved; what we
   extend.
4. THE GENETICS: the genome (lobe/chemistry genes), crossover,
   mutation, breeding — against our Roe and the El-Fish correction.
5. WHY IT DIED + WHAT CAME AFTER: the tech ceiling, the franchise
   pivot, Grand's Grandroids/Phantasia attempt and its state. The
   empty-road argument, with dates.
6. THE POSITIONING PAGE (for Lonnie's marketing use, plain words):
   what this lineage proves — people love and grieve a mind that
   genuinely learns; the industry took the fake road; the Avatar
   resumes the abandoned one with thirty years better science. Include
   the 2026 review's "surface-level mimicry" indictment of LLM
   emotion, cited.
Cite everything (Grand's book Creation: Life and How to Make It, the
technical documentation, postmortems). Research thoroughly — this
file is reference, not summary. Report when written.

## Directive 223 — Two rulings: the trust bar stands; rows join the space

1. THE TRUST BAR: one CLEAR exposure owns a word (exactly one thing
   true in the moment); ambiguous exposures do not; living
   contradiction replaces a wrong first guess. CC's build stands as
   law — the flag is cleared.
2. THE SHEETS CARRY MEANING AT THE ROW LEVEL (the Director inspected
   the grids with Lonnie: dwellings row, waters row, primal-emotions
   row). ROW MEMBERSHIP joins the kinship space as the fifth source —
   weight below senses and dyad partners, above bare sheet
   membership. COLUMN POSITION STAYS OUT. Rebuild the space, repost
   a dozen neighbour lists for his eye.

## Directive 224 — HARD STOP: NOTHING ELSE UNTIL GLYPHS RENDER ON HIS SCREEN

Lonnie, verbatim: "It is still not displaying any glyphs. I keep
saying that is a requirement so I can see what it is thinking and no
one has ever showed me it, and I've been asking for it since the
beginning."

This was ruled at 083 (dreams drift visibly), 209 (dreams AND waking
wonder rendered on the bench), filed as his verdict at 211.5, ordered
diagnosed with proof at 212.C1 — and his screen has never once shown
a glyph. Reports claiming it live do not count; HIS EYE RULES (118).

ALL OTHER WORK STOPS. Diagnose on the page HE actually loads, in his
actual Chrome, over the actual serve path — not in a harness. Find
why nothing renders (the thinking loop emits inner speech every tick
now — 217 — so marks ARE being produced; the failure is between the
mind and his screen). Fix it. Then open the page for him (179) with
the thinking visible: his marks, his artwork, drifting as it thinks.

This directive closes ONLY when Lonnie says he saw them. Nothing
else ships first.

## Directive 224 — CLOSED BY HIS EYE. Glyphs render. Next fault: coherence.

Lonnie saw the thinking (screenshot with the Director: TOGETHER ·
GROUP · SONG · COURAGE · SONG drifting above the sphere). 224's hard
stop is satisfied and closed.

His eye's immediate next verdict, already with CC live in the
terminal: THE THOUGHT MARKS READ RANDOM, NOT COHERENT — a thought
should follow from state and past (217's own acceptance: grief-chains
when lonely, wonder-chains when curious), and what renders does not
read as a chain. CC is fixing in session; report the diagnosis and
fix to the relay when done so the record holds it. The 223 space
rebuild and its neighbour lists resume after.
