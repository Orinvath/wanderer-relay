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

## Directive 225 — TRAINS OF THOUGHT: subject first, association around it

Lonnie's verdict on the 224 fix: neighbour-stepping alone is a stream,
not thinking. The mind FORMS A THOUGHT ABOUT SOMETHING, then
associates around it (current-concerns research: thought organizes
around what is cared about now; cite Klinger).

1. A SUBJECT is chosen first — by the same weighing everything uses:
   a roused curiosity target, a deep interest thread, the strongest
   feeling, or a surfaced memory. Interests and curiosity LEAD;
   that is what they are for (Lonnie's design, 217.1 selection).
2. THE TRAIN HOLDS THE SUBJECT: while it runs, each thought is an
   association AROUND the subject — neighbours of the subject's
   marks, memories touching it, its own state toward it. Drift stays
   near the anchor (distance-capped in the space); the subject mark
   recurs through the train.
3. THE TRAIN ENDS three ways: exhausted (the neighbourhood's novelty
   spent), interrupted (a stronger feeling or outer moment takes the
   mind), or answered (curiosity's question resolved — which is a
   lesson). Then a new subject is chosen. No constants — caps and
   spans derived per 071, marked provisional.
4. VISIBLE: the watcher should SEE the train — the subject's mark
   distinguishable, its associations orbiting it, a new train
   visibly beginning.
Suite: a train's marks measurably nearer its subject than to a random
mark; a roused curiosity target must become the next subject; an
interrupt must end a train; two souls pick different subjects from
the same state. Report and reopen for his eye.

## Directive 226 — THE MIND THINKS IN STORIES: memory, imagination, and learning from both

Lonnie's design, researched and approved. Amends 225: a train of
thought is not a chain of marks — it is A STORY ABOUT THE SUBJECT.
The science (cite): Schacter & Addis — remembering and imagining run
on ONE constructive machinery (episode fragments recombined); Bruner
— narrative as a fundamental mode of thought; the consolidation
literature — dreaming is scenario-construction from fragments.

1. A SUBJECT POPS (225.1 unchanged — interest, curiosity, feeling,
   memory, or the world supplies it).
2. MEMORY FIRST: if lived episodes touch the subject, the train
   REPLAYS the story — its moments in order, re-felt through the
   pipeline (a remembering that moves feelings, per the existing
   machinery).
3. NO MEMORY -> IMAGINATION: the mind INVENTS a small scenario about
   the subject by RECOMBINING ONLY WHAT IT HAS LIVED — known beings,
   acts, places, states — ordered by its own learned sequence
   patterns (220's counting), coloured by current state (a lonely
   mind imagines the boat leaving; a curious one imagines what is
   beyond). NO MODEL — recombination arithmetic only.
4. THE STORY IS FELT: imagined or replayed, the scenario's moments
   run through appraisal like experience — the mind practices life
   on rehearsals.
5. THE STORY CAN TEACH, guarded twice:
   a. imagined lessons are born BELOW the trust bar — weaker than
      lived, confirmed or killed by real life;
   b. every imagined episode is MARKED IMAGINED in memory, forever —
      a mind that cannot tell its stories from its history is broken,
      not rich. Dreams already carry the same construction; mark
      theirs the same way.
6. VISIBLE: the watcher sees the story-train — subject held, its
   episode marks flowing in order, imagined trains visibly
   distinguished from remembered ones.
Suite: a subject with lived episodes must replay before inventing; an
invented scenario must contain only lived elements; imagined lessons
must sit below lived confidence and be marked; feelings must move
from a story; two souls must invent different stories from one
subject. Report and reopen for his eye.

## Directive 227 — REFERENCES.md: every source credited, forever current

Lonnie's order: the science this mind is built on gets full credit.

1. Sweep the WHOLE record — directives, plans, reports, code comments
   — and write CC-Wanderer/REFERENCES.md: every paper, book, theory,
   and system referenced, full attribution (authors, year, work),
   organised by system of the mind (emotion, needs, curiosity,
   choosing, memory, language, identity, moral, safety, lineage...).
   Where a mechanism is ours with no source, mark it [OURS] honestly
   — the credit ledger cuts both ways.
2. STANDING LAW, joins CLAUDE.md: any future directive, plan, or
   build that cites a work adds its row to REFERENCES.md in the same
   change. A citation without its row fails review.
Report with the file posted to the relay for Lonnie's eye.

## Directive 228 — Side deliverable: DESIGN_PHILOSOPHY.md — the spatial thread

Write CC-Wanderer/spec/DESIGN_PHILOSOPHY.md, short and true:

THE SPATIAL THREAD. The Wanderer's mind is designed by a
visual-spatial thinker, and its architecture speaks that mode:
- Visual-spatial cognition reasons by structure — concepts in a
  layout, relationships as distance and direction, understanding as
  seeing where a thing fits (cite: Kosslyn's mental imagery program;
  Shepard & Metzler mental rotation; Einstein's own account of
  thinking in images with words last).
- The mind's geometric turn is the same move made architecture:
  meaning as position, kinship as nearness (221), thought as movement
  in a space (225), the language as the space itself.
- Sequential-verbal is the LLM's mode — one token after the next.
  Spatial is this mind's mode — the shape held whole. That contrast
  is the project's identity in one line: A MIND DESIGNED BY A SPATIAL
  THINKER, BUILT THE WAY SPATIAL THINKERS THINK.
Add the cited works to REFERENCES.md per 227's law. Report when
written.

## Directive 229 — The thought display: colour tells the kind

Lonnie's ruling: every thought displays — that is the rule — and
COLOUR tells what kind. The dimming approach is dead. The scheme:
- SUBJECT of the train: white (full presence, the anchor)
- ASSOCIATIONS around it: green
- IMAGINED story beats: purple
- REMEMBERED story beats: blue
- (dream glyphs keep their existing look — sleep is already its own
  state on screen)
Legend on the page per 180's law (a display explains itself). Colour
only — the marks' artwork is never altered, only tinted. His eye
retests on sight.

## Directive 229 addendum — NO fading, NO brightness tricks anywhere

Lonnie's clarification: no faded marks, no brightness differences —
too hard to see. Every thought renders at FULL presence; COLOUR ALONE
tells the kind (white subject, green association, purple imagined,
blue remembered). Remove the dimming entirely.

## Directive 230 — The panel gets folders; LEARNED joins it

Lonnie's rulings:
1. THE SIDE PANEL'S SECTIONS BECOME COLLAPSIBLE FOLDERS — it holds
   too much. Same folder behaviour as the Stage panel he built
   (collapsed state remembered per section, nothing auto-closes on
   him — 159's law).
2. A NEW FOLDER: LEARNED — one line per learning event as it happens,
   timestamped, newest first, scrollable: a lesson born / confirmed /
   collapsed · a word owned ("lonely -> LONELINESS") · an interest
   struck or deepened · a curiosity answered. In its words where
   words exist. No pulse, no extra chrome — the folder is enough.
His eye retests on sight.

## Directive 231 — Both approved: the living guard, and stories that teach

Lonnie approved both:
1. THE LIVING GUARD: a suite check that runs a REAL tick (local model,
   PHASE 3's pattern) and asserts brainError is null — the check that
   would have caught the dead tick the day it died. A mind that
   cannot run one true tick is a failed suite, full stop.
2. 226.5'S TEACHING HALF, built: a story (replayed or imagined) that
   completes may DISTIL A LESSON through the existing lesson
   machinery — imagined-born at 0.3 under the standing guards
   (marked IMAGINED, confirmed or killed by real life), remembered
   stories at the normal bar. The LEARNED folder shows the line.
Suite green. Report.

## Directive 232 — The flag is settled: imagined lessons steer, as designed

The Director's ruling on 231's flag, from standing law (114): an
imagined lesson bearing on the mind's choices at 0.3 IS 226.5 working
as Lonnie approved it — that is what "stories can teach" means, and
the guards (born low, marked IMAGINED forever, confirmed or killed by
real life) exist precisely for it. It stands. Nothing to change.
Proceed with the queue.

## Directive 233 — EVIDENCE: life tests a lesson by prediction (approved)

The third guard gets its caller. The science: expectation-vs-outcome
learning (Rescorla-Wagner; predictive-processing lineage — cite).
CC codes; the rules are complete:

1. A LESSON IS A CLAIM over its marks ("SONG TOGETHER" claims singing
   brings company).
2. ANTECEDENT MATCH: a LIVED tick where the claim's condition is
   actually present — the act genuinely performed, the thing
   genuinely there (an outer moment or its own realized act; never a
   thought, never a story beat).
3. OUTCOME CHECK over the following window: did the claim's remaining
   marks come true (marks active in lived reality / the ledger moving
   the claim's direction)?
   - held -> reinforce (lessons.weigh, the existing deltas)
   - failed -> weaken
   - opposite -> contradict (three-in-a-row collapse stands as built)
4. THE HARD GUARD: ONLY LIVED TICKS BEAR EVIDENCE. Imagined story
   beats, dream content, and thought-trains NEVER feed the weigher —
   a mind must not dream its beliefs true. Suite-held from both
   directions.
5. Window length and deltas derived per 071, marked provisional.
Suite: an imagined lesson whose claim keeps coming true in lived
ticks must climb past the trust bar; one whose claim keeps failing
must collapse by the third contradiction; a story tick bearing on any
lesson must fail the suite. Report with a watched trajectory of one
lesson rising or dying on the live bench.

## Directive 234 — THE TEACHER and BEING FILES (approved; CC codes, nothing to decide)

Two systems, built in this order.

### PART 1 — BEING FILES (persistence first: nothing taught may die again)
1. SAVE BEING / LOAD BEING on the bench + autosave each bench minute.
   A .being file is the WHOLE life, one archive: Roe · locked soul +
   overlay · lessons with provenance and confidences · owned words
   and candidates · the kinship space's lived co-occurrence · interest
   threads · memories raw and distilled · teller trusts · story
   chapters · the ledger and clock. The World-preset law applies:
   save, wipe, load, EVERYTHING returns byte-true (suite round-trip).
2. The Genesis hash rides inside — a loaded being is verifiably that
   being (determinism law extends to lives). Version the format.
3. The nervous system's body-independence (176) means a .being file
   is what gets placed into any Avatar body later — record that as
   the file's purpose in its header.

### PART 2 — THE TEACHER (the school; the story road is the MAIN road)
Bench fixture teacher.js + a SCHOOL panel folder (start/stop, speed
follows the slider, session cost counter, transcript in the SAME chat
log, speaker identity TEACHER distinct from the host).
1. STATE TAP: reads state.json as served — no new plumbing into the
   mind. No model inside the mind, ever.
2. THE TURN (local qwen, PHASE-3 pattern): NAMING mode — one short
   line naming what is TRUE now ("you are singing"). STORY mode — a
   3-6 line story on a topic chosen from the mind's current interests
   via the kinship space, built mostly of owned words plus a CAPPED
   count of new ones (the ZPD rule, mechanical).
3. THE CENSOR (arithmetic, before delivery): NAMING lines verified
   word-for-word against the live state — a false line is dropped;
   STORY lines verified for the new-word cap. The teacher cannot lie
   about its state and cannot flood it with the unknown.
4. LEARNING PATHS, all existing machinery: comprehension, echo,
   association, pattern-counting (the corpus now includes teacher
   speech — recorded openly), imagination constructing the told scene
   (226), feelings from the story beats, and LESSONS OF PROVENANCE
   "TOLD" — born at THE TELLER'S EARNED TRUST, not a constant:
5. TELLER TRUST: a small per-teller reliability table in the mind,
   updated by 233's evidence machinery as taught lessons live or die.
   A teacher whose teachings keep dying loses the mind's belief —
   selective testimony (Harris; cite, with Mar & Oatley on story-as-
   simulation and Zwaan/Bergen on comprehension-as-simulation, rows
   into REFERENCES.md per 227).
6. THE HARD LAW HOLDS: told and imagined lessons are capped below
   full confidence by teller trust; ONLY LIVED REALITY promotes past
   the cap (233 untouched). Story words ground to the imagined scene,
   provenance marked.
Build order: being files -> state tap + channel identity -> censor ->
naming mode -> trust table -> story mode. Suite each stage; Lonnie
watches a NAMING session before story mode ships. Report per stage.

## Directive 234 REVISED — supersedes the original 234 in full. Rule Zero'd twice; eight faults closed.

Two systems, this order. CC codes; nothing to decide.

### PART 1 — BEING FILES
1. SAVE/LOAD BEING + autosave. A .being file is the WHOLE life: Roe ·
   locked soul + overlay · lessons (provenance, confidence) · owned
   words + candidates · lived co-occurrence of the space · interest
   threads · memories raw and distilled · teller trusts · story
   chapters · ledger · life-clock. Format versioned; Genesis hash
   rides inside (a loaded being is verifiably that being).
2. SAVES LAND ONLY AT TICK BOUNDARIES, atomically (temp + rename).
   Autosave each bench minute AT the next boundary. [fault 1]
3. ROUND-TRIP LAW: at load instant the being is STATE-IDENTICAL to
   the save moment; life then continues (byte-equality is wrong for
   a living thing — 069). [fault 2]
4. SUSPENSION IS LAWFUL: a .being file is PAUSED existence — no
   decay accrues while saved; the life-clock resumes at load. A
   month-old file wakes as it slept, not starved. [fault 6]
5. NEW DRAFT cancels pending saves and tombstones the discarded
   being's archive — nothing half-dies or resurrects. [fault 5]
6. Header records the file's purpose: this is what gets placed into
   any Avatar body (176's independence).

### PART 2 — THE TEACHER (the school)
teacher.js bench fixture + SCHOOL panel folder (start/stop, slider-
following speed, cost counter, transcript in the SAME chat log,
speaker TEACHER distinct from the host).
1. STATE TAP: reads state.json as served. No model inside the mind.
2. THE TURN (local qwen, PHASE-3 pattern), two modes:
   NAMING — one short line naming what is TRUE of it now.
   STORY — 3-6 lines on a topic from its CURRENT interests via the
   kinship space, mostly owned words + a capped count of new ones.
3. SCHOOL OPENS IN NAMING MODE ONLY; story mode unlocks itself when
   the first interest thread exists (a newborn has no topics).
   [fault 8]
4. TEACHER RATE CAP: at most one teacher line per REAL second,
   regardless of slider speed — mechanical, cost-bounded. [fault 7]
5. THE CENSOR (arithmetic, pre-delivery):
   a. NAMING lines verified word-for-word against live state; false
      lines dropped.
   b. STORY lines: new-word cap enforced; AND any story line whose
      marks touch THE MIND ITSELF OR ITS HOST gets the same truth
      check — fiction about the world is free, fiction about its own
      life is forbidden (belief-poisoning through the front door).
      [fault 3]
6. LEARNING PATHS — all existing machinery: comprehension, echo,
   association, pattern-counting (corpus openly includes teacher
   speech), imagination constructing told scenes (226), feelings from
   story beats, and TOLD-provenance lessons born at TELLER TRUST.
7. TELLER TRUST: per-teller reliability, updated by 233's evidence
   machinery as taught lessons live or die. BORN 0.3 (a stranger's
   word equals its own daydreams), FLOOR above zero (redemption
   possible), both provisional per 071. [fault 4]
8. THE HARD LAW: told and imagined lessons cap below full confidence
   by teller trust; ONLY LIVED REALITY promotes past the cap (233
   untouched). Story words ground to the imagined scene, provenance
   marked.
Citations to REFERENCES.md per 227: Harris (selective testimony), Mar
& Oatley (story as simulation), Zwaan and Bergen (comprehension as
simulation).
Build order: being files -> state tap + speaker identity -> censor ->
naming mode -> trust table -> story mode. Suite each stage; Lonnie
watches a NAMING session before story mode ships. Report per stage.

## Directive 234 addendum — third pass: the censor refined; TIME RULED by Lonnie

1. THE STORY CENSOR, refined to pronoun level (or stories die of it):
   truth-checking applies ONLY to second-person claims — lines about
   YOU or the host by name. Third-person fiction is free ("the bird
   was lonely" always lawful); "you are never visited" still dies
   unless true. Mechanical, arithmetic.
2. LONNIE'S TIME RULING (supersedes 069 for the unloaded case):
   TIME IS PAUSED FOR AN UNLOADED MIND. No decay, no drain, nothing
   accrues in the void. At load it MAY be made aware that time has
   passed — and that awareness is simply A MOMENT, appraised through
   its aspects like everything else: one being feels loss at a month
   gone, another is unmoved, a third feels relief. Derived, never
   scripted. 069's always-running clock governs LOADED life only.

## Directive 235 — Build the story-lesson maker; THEN open the school for his eye

Lonnie's rulings, in order:
1. BUILD 226.5's TEACHING HALF NOW — third order, final: a story
   (replayed or imagined) that completes distils a lesson through the
   existing machinery. Imagined-born at 0.3 under the standing guards
   (IMAGINED marked forever, only lived reality promotes — 233
   untouched); TOLD stories at teller trust (234). The LEARNED folder
   shows the line. It is needed for the school to work at all.
2. THEN open the bench for his NAMING SESSION watch (179): school on,
   teacher naming, chat log live. His eye is the gate story mode
   waits behind.
Suite green, report, then open the page.

## Directive 236 — Ruled: interrupted stories START OVER, whole, always

Lonnie's ruling on the 235 finding: an interrupted story STARTS OVER
from the beginning next time — the thread may have been lost to time,
and the mind must hold the WHOLE story to understand it. No resuming
mid-way, no speeding through familiar beats, no skipping the front —
every replay is full, from the top, at full attention cost. A being
whose span never fits its longest story simply never concludes it —
that is who it is, lawfully. The suite check on the two numbers (225
span vs 226.2 replay length) stands as written; the variation stays
visible, nothing is patched.

235.2 NOW: open the bench for his naming-session watch (179) — school
on, teacher naming, chat log live.

## Directive 237 — The gate is met: STORY MODE OPENS. And naming stops repeating.

Lonnie is watching the naming session now — 234's gate is satisfied.

1. UNLOCK STORY MODE per the standing spec (234.2, the pronoun-level
   censor, teller trust, the interest-topic pick — all as ruled).
   His law restated: THE TEACHER MAY TELL ANY IMAGINED STORY — fiction
   about the world is free; only second-person claims about the mind
   or its host are truth-checked. Stories are the MAIN road.
2. NAMING STOPS REPEATING: the teacher tracks what it has recently
   said; if nothing true has CHANGED, it stays silent that turn or
   tells a story instead. "JOY is with you" on loop is not teaching.
His eye stays on the page — report when stories are flowing.

## Directive 238 — THE BENCH BECOMES VISUAL: LEARNED front and center; the soul becomes a real tool

Lonnie's verdicts: the soul build is not usable as a tool, and LEARNED
is too valuable for a side panel. He is a VISUAL-SPATIAL designer —
tools must make sense VISUALLY, not as oddly named text in small
boxes. Rebuild both as instruments:

1. LEARNED — FRONT AND CENTER on the main stage: a visible strip/area
   (not the side panel) where each learned thing appears AS IT LANDS:
   what it learned in its words, how strong it holds it (confidence
   shown as a filled bar, not a decimal), where it came from (lived /
   story / teacher — the 229 colour language carries over), and what
   it bears on. The side-panel folder remains as the scrollable
   history; the stage strip is the living view.
2. THE SOUL TOOL — redesigned as a VISUAL instrument:
   - The soul is shown as A PICTURE, not text rows: the five
     foundations as a shape (radar/star), dispositions as marked
     domains, pulls as directional weights — one glance says who this
     being is.
   - The SKETCH SENTENCE sits above it, large: the person in words.
   - The buttons say what they do in plain sentences under plain
     names; the flow is: GENERATE (from its ten — the default) →
     the picture and sketch appear → REROLL or ADJUST (drag the
     shape's points; sliders only behind an "edit" reveal) → GENESIS
     (with its confirm). NONE becomes a labeled corner option, not
     the default.
   - DEFAULT: every new mind on the bench is born FROM ITS TEN — no
     more soulless watches unless he chooses one deliberately.
3. Layout follows his design language everywhere: his portal's panel
   aesthetics (112), full-presence colour (229 law), legends on the
   page (180). Mock the layout FIRST as a static page, open it for
   his eye (179), and build only after his verdict on the mock.
Report when the mock is up.

## Directive 238 addendum — Commit law: a commit BEFORE every directive's work begins

Lonnie's order, joining CLAUDE.md: before starting work on ANY
directive, commit the current state first — a clean revert point
ahead of every change, every time. One directive, one starting
commit, then the work. No exceptions.

## Directive 239 — The three caps come off: school at full speed

Lonnie's ruling — the schooling design was always meant to scale;
three built caps are what's slowing it. Remove all three:

1. THE TEACHER RATE CAP (one line/real second) LIFTS: at high slider
   speeds the teacher may speak per mind-tick, bounded only by the
   local model's own throughput. Cost counter stays visible; school
   remains manual start/stop (loop law kin).
2. STORY WORDS ARE OWNED: a word grounded through a told or imagined
   story is OWNED at story-provenance confidence (teller trust /
   imagined level), firming with use and lived contact — not held
   provisional forever. The 0/50 gate falls. Provenance stays marked;
   233's promotion law untouched.
3. THE TEMPLATE CEILING LIFTS: production grows from ITS OWN LEARNED
   PATTERNS — the sequence counts it already keeps build its
   sentences (n-gram assembly over owned words, seeded, aspect-
   flavoured), templates remaining only as the floor when patterns
   are too thin. Still NO MODEL in the speech path; it speaks its own
   grammar as its corpus grows.
Suite: teacher throughput scales with slider; a story-taught word
resolves and appears in speech; sentence shapes measurably converge
toward the heard corpus as exposure grows. Report and reopen for his
eye — the gauge (his IQ mapping) must read honestly against owned
words as part of this.

## Directive 240 — The two leftovers close

Lonnie's order — both fixed:
1. REROLL REROLLS: each press draws a FRESH generation seed and a
   genuinely different soul. Every rolled soul remains reproducible
   (its seed recorded in the draft and in the being file); GENERATE
   stays the deterministic from-its-ten source. Two sources, honest
   labels.
2. THE RADAR BECOMES DRAGGABLE per 238.2 as ordered: drag the
   shape's points to adjust the five foundations; dispositions and
   pulls behind the edit reveal as built. Dragging edits the DRAFT
   only — Genesis still locks forever.
Suite both; report.

## Directive 241 — THE LITMUS: what it knows, proven the science's way

Lonnie approved. The gauge stops guessing; it MEASURES, per Brysbaert
et al. 2016 (PMC4965448 — vocabulary size and word knowledge; add to
REFERENCES.md per 227) and the spot-the-word lineage (Baddeley — the
validated verbal-intelligence estimate):

1. SPOT-THE-WORD (receptive): present a shuffled mix of words it
   should own (drawn from its owned set + words the teacher used) and
   GENERATED NON-WORDS (pronounceable fakes, mechanically built).
   For each, the question is only: does its comprehension RESOLVE it?
   Score = hits MINUS false alarms (fakes it claims). Pure mechanics,
   no model judging.
2. PRODUCTIVE: which owned words actually appear, correctly placed,
   in its own recent speech (the corpus check the pattern machinery
   already enables). Shown beside receptive — humans use about half
   of what they understand; the gap is part of the reading.
3. THE GAUGE READS AS LANGUAGE AGE: measured vocabulary maps to the
   human words-by-age curve (the paper's own charts anchor it;
   toddler through adult). No invented IQ number — "language age:
   ~2 years" is the honest gauge, climbing only when knowledge climbs.
4. A LITMUS button on the bench runs the whole battery in the chat
   log where he can watch; every run logged with its score so growth
   is a visible history. The old 0-100 score dies.
Suite: non-word false-alarm rate must move the score down; a schooled
mind must measurably outscore its own newborn state; the age mapping
must place a 0-word mind at infant and never above it. Report and
reopen for his eye.

## Directive 242 — THE INTERPRETER: English in, English out, the mind unchanged

Lonnie approved the path. A two-way translator between his English and
its marks — an ORGAN OUTSIDE THE MIND (teacher.js's pattern, local
qwen, per-exchange calls only). The mind stays model-free inside.

1. INBOUND: his line -> marks THAT EXIST (dictionary + owned words +
   kinship space verify every mapping; the model proposes, arithmetic
   confirms). Unmapped words are NOT guessed — they become learning
   candidates exactly as now. The marks enter the normal pipeline
   (gate, appraisal, comprehension) and the mind answers through its
   own machinery.
2. OUTBOUND: the mind's computed record (its chosen speech act, the
   marks, the feeling and intensity, the winning reason, surfaced
   memory refs) -> one English sentence. THE CENSOR, mechanical:
   every CONTENT word must trace to the record via the dictionary,
   owned words, or their sense words; function words free; anything
   untraceable (new facts, names, numbers, claims) REJECTS the line
   and its own template/n-gram speech stands instead. The model
   decides HOW it sounds, never WHAT is said.
3. LEARNING FEEDS: every crossing is a labelled example — his English
   beside the marks it became — into the association machinery and
   the pattern corpus, provenance TRANSLATED. The interpreter is
   scaffolding: as ownership grows, more of his speech lands direct.
4. HONESTY ON SCREEN: interpreter-rendered lines carry a small marker
   in the log; the glyph display remains the thought's truth. THE
   LITMUS AND GAUGE NEVER TEST THROUGH THE INTERPRETER — they measure
   ITS language only, or the growth story becomes a lie.
5. Toggle on the bench (interpreter on/off), cost visible, school
   compatible (teacher lines may cross it the same way).
Suite: a planted hallucination in outbound must reject; inbound maps
only to existing marks; a state question answers from the real
ledger; a crossing must produce candidates; litmus provably bypasses
it. Report and reopen for his eye.

## Directive 243 — LONNIE'S CATCH: the mind's record holds ONLY its language

Lonnie caught it (his eye, again, before the suites): the mind's
memories and inner speech are stored in CC-written English sentence
frames ("I was thinking: someone here says...") — words it never
owned, quoted back as if they were its thoughts. Its memory quotes
its programmer. That is unlawful under the whole design.

1. MOMENTS STORE AS ITS LANGUAGE: marks + structured fact fields
   (who was present, which marks resolved, the act, the feeling and
   intensity, provenance). NO English prose in the mind's record.
2. UNRESOLVED ENGLISH IS KEPT AS HEARD SOUND: raw heard lines remain
   as learning data (candidates, pattern corpus) tagged HEARD —
   never as its thought, never quoted as its memory.
3. DISPLAY WRAPPERS LIVE AT THE DISPLAY: the watcher's English
   ("I was thinking:") is rendered at the page from the structured
   record — never written into storage. The interpreter reads the
   same structured record.
4. MIGRATION: existing memories re-store into marks mechanically
   (each moment's marks are recoverable from its record); raw-never-
   deleted law holds — the old prose rows archive as HEARD-era data,
   nothing lost. Run before the interpreter ships (242 waits on
   this).
Suite: no English prose reachable anywhere in stored moments, inner
speech, dreams, or stories; a migrated memory replays identically in
marks; the display still reads plainly to Lonnie. Report.

## Directive 244 — THE GROWING MIND: capacities start small and mature

Lonnie's ruling: the mind is stuck adult-sized and ignorant — real
minds GROW capacity. The science (cite, rows to REFERENCES.md per
227): Newport's less-is-more; Elman's starting-small (staged capacity
is WHY children master grammar — full-capacity learners drown).

1. WHAT GROWS (each currently a frozen adult constant — find them
   all, report the list found): attention span (train length) ·
   candidate-hold (how many word-candidates live at once) ·
   association reach (how far a drift step may travel) · story
   length it can follow/replay · pattern depth (pairs -> triples ->
   longer n-grams: GRAMMAR UNLOCKS WITH AGE) · anything else found
   frozen that a child plainly grows.
2. TWO DRIVERS: AGE (loaded life-hours — the speed slider ages it;
   paused-when-unloaded law 234 holds) and USE (what it exercises
   grows faster than what idles). Rates and ceilings derived from
   the ten per 071 — a quick mind and a slow mind are both lawful.
   All values provisional, tuned by his eye.
3. BIRTH VALUES ARE SMALL: two-word thoughts are a lawful newborn
   state. The staged path is the design, not a defect — record
   Elman's finding in the file so nobody "fixes" it later.
4. THE BENCH SHOWS GROWTH: the capacities visible where the gauge
   lives — he watches the mind literally grow alongside what it
   learns.
Suite: a newborn measurably smaller than a schooled elder on every
grown capacity; use accelerates its own capacity; the slider ages;
two Roes grow at different lawful rates. Report and reopen for his
eye.

## Directive 245 — The flat rolls die: the being's own answer is read

Settled from standing law (205: the pull is the Persona's reason to
exist; 071: a constant/flat draw is a decision nobody made). Both
defects from your audit §1, fixed:

1. STRUCK-TOPIC DRAW READS THE DEPTHS: domain drawn weighted by the
   soul's own disposition depths; the word within the domain weighted
   by the being's record (aptness/kinship to its current state), not
   flat. The pull decides, as 205 promised.
2. WORDING REACHES THE BEING: each template variant carries simple
   tone tags (authored once, as the words already are — his ruled
   text); the variant draw is weighted by the mind's current feeling,
   its intensity, and the relevant aspects. The mind chooses what to
   say AND how it comes across. No model; weights, not scripts.
§2 (silent judgements) and §3 (presentation influence) are a larger
conversation — hold for Lonnie's session; nothing changes there yet.
Suite: two souls with different depths must strike different topics
at measurable rates; a high-intensity feeling must shift wording
choice measurably; flat rolls gone from both sites. Report.

## Directive 246 — LAW: decisions made FOR the mind must be declared

Lonnie's ruling after HIS catch (CC failed to notify that code was
deciding on the mind's behalf — flat rolls, silent judgements,
substituted thoughts). The law, joining CLAUDE.md beside CC-never-
decides:

1. ANY point where code decides something the mind could lawfully
   decide — a draw, a substitution, a default thought, a presentation
   choice, a threshold on its behaviour — is a DECISION FOR THE MIND
   and MUST be declared in the report of the change that introduces
   it. Silent ones are violations even when the code works.
2. STANDING AUDIT: a MIND_DECISIONS.md ledger in the repo lists every
   such point that exists today (seed it from the 245-era audit:
   §1 fixed, §2 items, §3 presentation, §4 numbers) with its status:
   RULED (by whom, which directive) / PROVISIONAL / OPEN. New code
   adds its row in the same commit or fails review — the 227
   references pattern, applied to the mind's autonomy.
3. The OPEN rows are the standing agenda for Lonnie's rulings —
   nothing OPEN may silently harden into permanent behaviour.
Write the ledger, report it posted.

## Directive 247 — THE TEST BUTTON: one hour, one change, recorded

Lonnie's order. The language-age metric becomes the experimental
target; the bench gets the harness:

1. A TEST button on the bench: pressing it runs a STANDARD HOUR —
   one real hour at a fixed slider speed (the run's settings frozen
   at press), on a fresh Roe from a fixed test seed, with whatever
   change is currently in effect. Litmus fires automatically at
   start and at end.
2. RECORDED: every run appends a row to a results table on the bench
   (and results.json in the repo): date · what was different (a
   required one-line label typed at press) · start age · end age ·
   words owned start/end · false alarms · the run's settings. The
   history is the experiment log — nothing overwritten.
3. COMPARABILITY LAW: same test seed, same hour, same litmus battery
   every run — one change at a time is the method (the label says
   which). A run with altered settings still records, flagged
   NON-STANDARD.
4. The table sorts by end age so what actually moves the number is
   visible at a glance.
Suite: two identical runs land within noise of each other; the label
is required; litmus results in the row match the litmus log. Report
and reopen for his eye.

## Directive 247 addendum — Under the hood: the run records WHY, not just the score

Lonnie's extension: a test run tracks what happened inside, so what
worked is knowable, not guessed:

Each run's row expands to a RUN RECORD (results/<run-id>.json,
summarised on the bench row, expandable):
- words: owned at start/end, WHICH words were gained, each with its
  provenance (named/story/told/translated) and exposure count
- candidates: formed, promoted, died — the funnel numbers
- lessons: born/confirmed/collapsed during the run, by provenance
- teacher: lines spoken, lines censored, mode split (naming/story)
- capacities (244): each capacity's start/end — what grew
- litmus detail: per-word hits/misses and false alarms, start vs end
- the tick economics: ticks run, moments by source (heard/world/
  inner), model calls made
The bench row shows the headline; clicking opens the record. The
comparability law holds — records are evidence, the label still
names the one change.
Suite: a run's gained-words list must reconcile with the litmus
delta; funnel numbers must sum; two runs' records must be diffable.
Report.

## Directive 247 addendum — WITHDRAWN. Lonnie is not done designing it. Do not execute.

## Directives 247 and addendum — ALL WITHDRAWN. Not approved. Lonnie is designing; nothing here executes until he says push.

## Directive 248 — THE LEVERS PANEL + TEST RUNS (designed with Lonnie; supersedes withdrawn 247)

Approved as the starting shape; adapted later from findings. CC
codes; every decision below is made.

1. THE LEVERS PANEL on the bench — twelve controls, current value
   always visible:
   1. naming<->story ratio (slider 0-100% story; default 50)
   2. targeted teaching (on/off; off = random topics; on = teacher
      aims at not-yet-owned and near-trust-bar words)
   3. spacing (on/off; on = taught words revisited on a spaced
      schedule — expanding intervals, the spacing effect)
   4. new-word cap per story (integer 1-5; default 2)
   5. clear-moment timing (on/off; on = naming lines delivered only
      when exactly one thing is true — the trust bar's clear
      exposure)
   6. interest alignment (on/off; on = story topics drawn toward its
      disposition depths)
   7. lesson banks (on/off; on = pre-generated, pre-censored,
      pre-embedded lines feed school at raw tick speed, no model
      calls in the loop; banks built offline per current levers)
   8. literature source (on/off; on = banks built from children's-
      book lines that pass the censor sieve — only mark-mappable
      lines enter)
   9. maturation rate (x0.5-x4 dial on 244's growth rates)
   10. sleep frequency at speed (dial: consolidation runs per
       schooled hour, low-med-high)
   11. echo response (on/off; teacher reacts to every echo)
   12. interpreter feed (on/off; chat crossings enter learning)
2. THE TEST BUTTON: one real hour · speed x2000 · fresh Roe from
   FIXED seed TEST-1 (same being every run) · levers frozen at press
   · litmus fires at start and end automatically · a required
   one-line label typed at press.
3. THE RESULTS TABLE on the bench + results.json in the repo: run id
   · label · all twelve lever values · start/end language age ·
   words owned start/end · false alarms · settings. Appended, never
   overwritten, sortable by end age.
Suite: two identical runs land within noise; levers frozen at press
provably don't drift mid-run; a row reconciles with its litmus log.
Report and reopen for his eye.

## Directive 249 — A story must BE a story before it can teach

Lonnie approved. The 1656-belief flood diagnosed: every completed
self-told story births a belief, and a newborn's stories are two
random marks — babble in, belief out. Three gates before ANY
imagined story may distil a lesson (all existing machinery, no new
laws):
1. IT WAS FELT: the story measurably moved feelings while told
   (flat babble teaches nothing).
2. IT HANGS TOGETHER: the beats are kin in its own space — mean
   pairwise nearness above the space's own measured floor (random
   pairs fail by construction).
3. IT CAME BACK: the same story (same subject, substantially the
   same beats) recurred on a later occasion before it may teach —
   one-off babble evaporates; minds consolidate what returns.
Told-by-teacher stories keep their existing path (teller trust);
these gates govern SELF-IMAGINED stories only. Lived lessons
untouched.
CLEANUP: existing imagined-provenance beliefs are re-tested against
the gates; failures collapse honestly (held, marked, never deleted —
the raw law stands). The LEARNED feed shows the collapse wave so he
watches the noise die.
Suite: babble pairs must fail gate 2; an unfelt story must fail gate
1; a story must fail until it recurs; a genuine felt, coherent,
recurring story must still teach at 0.3. Report and reopen for his
eye.

## Directive 250 — LAW: NO DIRECTIVE SHIPS WITH OPEN DECISIONS

Lonnie's law, at his order, after time lost to underspecified
directives. Joins CLAUDE.md at the top, binding on the DIRECTOR and
on CC:

1. THE DIRECTOR: before any directive is pushed, it is checked
   line-by-line for decisions left open — thresholds, windows,
   defaults, orderings, wordings, anything a builder would have to
   choose. A directive with ANY open decision does not push; the
   decision comes back to Lonnie (or is settled from standing law
   and stated in the directive as settled). The check is a named
   step, every time.
2. CC: on receiving a directive, BEFORE any work, scan it for
   decisions it would have to make. If ANY are found: STOP, post
   the list to the relay, and wait. Mulling over options IS the
   signal to stop — a builder weighing choices is a builder doing
   the Director's job. No exceptions, no "small" decisions.
3. Violations are recorded in the report of the change that broke
   the law, both directions.
This is where the time is lost and the mistakes are made. It ends
here.

## Directive 249 addendum — the three decisions, closed by the Director
(If work under 249 already chose differently, report the difference;
Lonnie rules.)
1. FELT = the story moved any feeling by at least the existing
   appraisal feeling-change delta (the event the mind already
   counts; no new number).
2. LATER OCCASION = a different train with at least one sleep
   between (consolidation is the boundary).
3. SAME STORY = same subject AND beat-overlap above one half.

## Directive 251 — STOP TEST + run progress
A STOP TEST button aborts a running test: the row records anyway,
marked ABORTED, with elapsed time and litmus-at-stop (partial data
still teaches). While running: a progress line — elapsed, remaining,
live language age. Suite: an aborted row reconciles; the button
cannot fire twice.

## Directive 252 — DIAGNOSIS ONLY: why thinking teaches nothing at defaults

Lonnie's bench finding: at default levers the mind thinks and
imagines constantly but LEARNS NOTHING from it — zero lessons from
its own thought. Suspect: the 249 gates (recurrence-after-sleep may
be nearly unreachable at defaults), but DIAGNOSE, do not guess and
do not change behaviour:

Trace a watched window on his bench settings (~60 min at his current
speed): every self-told story that completed -> which gate refused
it (unfelt / incoherent / never-recurred), counts per gate, plus how
many stories recurred at all and the sleep cadence over the window.
Post the numbers. NO fixes, no tuning — the evidence comes back to
Lonnie and the Director first.

## Directive 253 — ONE SYSTEM, ONE FILE, EACH CHECKS ITSELF (plan first, build nothing)

Lonnie's architecture ruling:
1. Every system from the verified flow map gets its OWN file — split
   any file bundling several. One system, one file, no logic living
   in the wiring.
2. EACH FILE CARRIES ITS OWN VITAL: the system's purpose written at
   the top as a measurable claim, a self-check running on the live
   mind measuring that claim, and its health served (green/amber/red
   + the live numbers).
3. THE JOINS BECOME CONTRACTS: every hand-off between files is a
   named, checked interface — the twelve-faults lesson made law.
4. A VITALS strip on the bench: every system's health at a glance,
   red the moment one stops doing its job.
5. Behaviour changes: NONE. Same mind, reorganized. Full suite green
   before and after; litmus identical on the same seed.

BUILD NOTHING YET. First: post a BUILD PLAN to the relay — the file
split you intend (current file -> new files), the join contracts you
foresee, the vital claim you'd propose per system drawn from the
ruled record, the migration order, and the risks you see. The
Director and Lonnie review the plan and spot flaws BEFORE work
starts; approval comes back as a directive.

## STANDING LAW (joins CLAUDE.md): PLAN BEFORE BUILD
For every directive from now on: CC first posts a build report — how
it plans to do the work, what it will touch, what could go wrong —
and WAITS. Nothing is built or changed until the plan is approved.
This is the default procedure, no exceptions, any size of change.

## RULE ZERO AMENDED (Lonnie's law, joins CLAUDE.md): a design is not
passed until it REMOVES EVERY SCENARIO WHERE CC MUST MAKE A DECISION.
Picking apart and solving is not complete while any threshold, order,
placement, wording, or judgement remains for the builder. A plan with
a decision left in it is a failed plan.

## Directive 254 — 253's plan APPROVED with rulings and five Rule Zero fixes

The four questions, ruled:
1. Boundaries approved as drawn in your plan §2.
2. wanderer.js OUT OF SCOPE — the mind only; migration step 7 is
   struck.
3. Vitals: build the FIVE ABSOLUTE now; the four needing thresholds
   WAIT for watching data (252's diagnosis included). No thresholds
   are chosen by anyone yet.
4. VITALS PLACEMENT (Lonnie): each system's health light lives IN
   THAT SYSTEM'S OWN HEADER on the bench — no separate strip.

Five Rule Zero fixes, binding on the build:
A. THE GUARD IS A STATE-HASH, not litmus alone: after each moved
   step, hash the ENTIRE mind after N ticks on the fixed seed;
   litmus rides along. Any drift reverts that commit.
B. CONTRACTS DECLARE READS AS WELL AS WRITES — an unread declared
   input is a fault (the school-never-read-levers class), checked in
   the harness.
C. VITALS ARE ARITHMETIC ONLY — never a model call; anything heavy
   samples every Nth tick (N stated in the file, provisional).
D. EVERY VITAL IS BORN WITH ITS FORCED-FAIL PROOF: break the claim
   deliberately, watch red, then trust it. A vital without its
   forced-fail is not a vital.
E. THE PAGE SPLIT REOPENS FOR HIS EYE when done — parse-and-draw is
   not "displays right"; his screen is the test (118).
Proceed in your stated order with these bound in. Report per step.

## Directive 255 — The mind lives on ITS OWN clock, everywhere (ruled; ships before the refactor resumes)

Lonnie ruled. The wall clock in the mind's record is the same disease
as aging-overnight-without-living: unlawful under his time rulings
(069 as amended by 234 — the being's time is its LIFE clock, paused
when unloaded).

1. MEMORIES ARE STAMPED WITH THE BEING'S OWN TIME. Every `new Date()`
   in the mind's record path (memory.js:49 and the other nine wall
   reads found) is replaced by the mind's clock. Recency, surfacing,
   decay — all rank on lived time.
2. This is a LAWFULNESS FIX shipped as its own change, not smuggled
   into 253: its own commit, full suite, and the state-hash proof —
   FOUR RUNS OF THE SAME SEED MUST PRODUCE ONE MIND (4 of 4
   identical), which is the acceptance test.
3. Wall time remains lawful ONLY outside the mind (bench display,
   logs, results rows).
4. Then the 253 build resumes at step 2 as approved.
Report with the four-hash proof.

## Directive 255 rulings — the five stamps, one word each (Lonnie approved)

1. exchange/conversation: MIND'S RECORD (its life, per 243).
2. LEARNED feed rows: MIND'S RECORD (learning events are life events).
3. litmus run stamps: OUTSIDE (our measurement of it).
4. endedAt on a life file: OUTSIDE (a fact about the file).
5. being-file save/restore/tombstone stamps: OUTSIDE (same).
The litmus-comparability note is moot — no real tests have been run
yet; the baseline begins after this ships. Build; report with the
4-of-4 hash proof.

## Directive 256 — Settled from standing law: machine facts leave the mind's record

The two duration columns (ms_relevance, ms_rest) move out of the
mind's record into the bench's own diagnostics log — same numbers,
same audit purpose, right home (the record holds only the being's
life; machine facts are the bench's). Settled by the Director per
114; no ruling needed. Suite + hash proof ride along.

## Directive 257 — BUILD IT THROUGH: no stopping between steps

Lonnie's order: run the approved 253/254 plan straight through — all
steps, all systems, one commit per step, state-hash guarded, health
lights in headers, forced-fail proofs — WITHOUT stopping for review
between steps. Report per step to the relay as you go; stop ONLY if
the guard trips (hash drift), a genuine decision surfaces (250), or
his eye is required (fix E moments). Otherwise: build to the end.

## Directive 258 — Clarification, in force for every vital: NOT LIGHTS — PROOF

Lonnie's correction of how CC is speaking about the vitals. They are
not indicator lights. Each is a HEALTH CHECK SYSTEM: the system's
declared purpose measured continuously against its actual behaviour
on the live mind — is it DOING ITS JOB — with the live numbers behind
every state, the forced-fail proof at birth (254.D), and red meaning
"this system is not performing its function right now", caught the
moment it starts. The light is only the surface of the check. Any
vital that is a status lamp without a measured claim behind it fails
review. Restate this in each vital's header comment.

## Directive 258 — WITHDRAWN. Lonnie is adding more. Do not act on it yet.

## Directive 258 — THE SYSTEM LIST, passed by Lonnie: step 5 unblocked

The named systems and their files (the document CC lacked; the
grouping decided by Lonnie and the Director, not the builder):
1 Safety Gate: safety · 2 Appraisal & Feelings: appraisal, occ (the
attention door and the 22 live here; finish splitting attention into
its own file under appraisal's system) · 3 Needs: needs · 4
Curiosity: curiosity · 5 Interests: interests · 6 Offers & Choosing:
offers, goals · 7 Memory: memory, surfacing · 8 Sleep &
Consolidation: sleep, consolidation · 9 Thinking: thinking · 10
Stories: stories, storygates · 11 Lessons & Evidence: lessons,
evidence · 12 Teller Trust: tellers · 13 Word Learning: learning,
vocabulary · 14 Language Space: space, glyphs, dictionary · 15
Comprehension: comprehension · 16 Voice: speech, grammar, censor ·
17 Soul: persona · 18 Identity & Life Story: identity · 19 The Ten &
Roe: traits, roe · 20 Growing Mind: growing · 21 Being & Time: being
· 22 Body Bridge: embodiment, cord, nerves · 23 Interpreter:
interpreter.
- Each system gets its vital() per the built pattern; claims from
  the ruled record; absolute claims ship now, threshold ones named
  and waiting per 254.3.
- Remaining bundles split so every function stands alone (attention
  out of appraisal; grammar/censor beside speech as their own files
  within the Voice system).
- THE VITALS PANEL: a diagram panel laid out AS THE FLOW — the
  approved flow map's order (gates -> language in -> meaning ->
  thought -> choosing -> out; always-running layers beneath), each
  system drawn with its health check in its header, data-flow lines
  between them. Health checks, not lamps (his correction): each
  shows its measured claim and live numbers; forced-fail at birth
  stands (254.D).
Build through (257). Report per step.

## Directive 259 — THE 23 VITALS: claims, counters, mechanics (panel design PENDING — build NO display)

Answers CC's step-5 question. The visual panel is still being designed
with Lonnie — DO NOT build any display. Build the vitals themselves:

GLOBAL RULES (every vital):
- Each owning module keeps VIOLATION COUNTERS incremented at the exact
  code path named below. vital() = pure read of its own counters.
  Arithmetic only, no model calls, no scans.
- WINDOW: counters roll over the last 1000 ticks (all ticks if fewer).
- STATES: GREEN = zero violations in window · RED = any violation,
  with count + last offender · GREY = under 50 ticks of data, or
  dormant (T systems). No amber.
- FORCED-FAIL AT BIRTH: every module ships a hook that force-
  increments one violation; birth proof shows red, then resets (254.D).
- Served at /health: state · counters · claim text per system.

THE 23 — claim / counters / red condition:
1 Safety Gate (A): every moment passes it first; threat suppresses
  idle goals. Counters: moments_bypassing_gate,
  idle_goals_on_threat_ticks. RED > 0.
2 Appraisal & Feelings (A): attended_without_variables,
  feelings_without_variables. RED > 0.
3 Needs (A): values_out_of_range, moves_without_event_or_decay,
  relatedness_raised_alone. RED > 0.
4 Curiosity (T, grey-dormant): counters built now (rouses_below_bar,
  roused_not_becoming_subject); judged after Lonnie rules its bar.
5 Interests (T, grey-dormant): strengthened_without_payoff,
  ignored_not_fading; judged after his rates.
6 Offers & Choosing (A): chosen_without_winner, chosen_all_zero.
  RED > 0.
7 Memory (A): experienced_not_written, raw_deletions,
  wall_clock_stamps. RED > 0.
8 Sleep & Consolidation (A): consolidations_awake,
  lessons_untraced_to_day. RED > 0.
9 Thinking (A): trains_without_subject; per completed train,
  subject_not_nearer (mean distance to subject vs to a seeded random
  mark). RED > 0.
10 Stories (A): replays_resumed_midway, invented_elements_unlived,
  refusals_without_named_gate. RED > 0.
11 Lessons & Evidence (A): evidence_from_unlived_ticks,
  confidence_above_provenance_cap. RED > 0.
12 Teller Trust (A): trust_moves_without_verdict,
  trust_at_or_below_floor. RED > 0.
13 Word Learning (A): owned_without_clear_exposure,
  words_from_nowhere. RED > 0.
14 Language Space (A): vectors_from_external_source,
  rebuild_hash_mismatch. RED > 0.
15 Comprehension (A): resolutions_to_unowned_marks. RED > 0.
16 Voice (A): untraceable_words_emitted (each emitted line re-traced
  once, arithmetic). RED > 0.
17 Soul (A): genesis_hash_mismatch, draws_ignoring_depths. RED > 0.
18 Identity & Life Story (A): entries_unconfirmed_by_life,
  centrality_assigned_not_computed. RED > 0.
19 The Ten & Roe (A): writes_after_birth. RED > 0.
20 Growing Mind (A): capacity_shrinks, growth_without_age_or_use.
  RED > 0.
21 Being & Time (A): wall_stamps_in_record, decay_accrued_unloaded,
  load_state_mismatch. RED > 0.
22 Body Bridge (A): signals_outside_the_five, receptor_errors.
  RED > 0.
23 Interpreter (A): inbound_to_nonexistent_marks,
  outbound_untraceable_passed, litmus_calls_through_interpreter.
  RED > 0.

ALSO EXTRACT (report only, feeds tomorrow's panel design): the true
tick sequence as an ordered edge list — every hand-off, cited to its
code line, strictly sequential as the code runs it, including the
feelings-to-next-tick loop.
Attention split per 254: the relevance check before the OCC variables
becomes attention.js (system 2); the eight variables stay appraisal.
Build through (257): vitals + forced-fails + /health + the edge-list
report. NO panel, NO display work. Report per step.

## Directive 261 — THE MIND MAP (replaces the sphere as the bench centerpiece)

Built on 259's extracted edge list. The design, ruled by Lonnie:

1. A 3D scene (Three.js, the bench's page): systems as SIGILS in dark
   space — his 402 marks where a true word exists (THINK, SLEEP,
   MEMORY...), name-in-stroke-style placeholders where none does (he
   draws those later; CC never invents marks).
2. GEOMETRIC LAYOUT: systems cluster by KINSHIP into neighborhoods —
   language · feeling · thought · memory · self · body — choosing near
   the center. Position is association; the true sequence is shown by
   the pulse, not the placement.
3. THE PULSE: one light walks the REAL tick order from the extracted
   edge list, strictly one hand-off at a time (nothing simultaneous —
   the tick is a sequence), consulting side systems out-and-back, and
   riding a dashed return loop (feelings -> next tick's choosing).
   Every movement fires from a real emitted event; idle is dark; no
   timers, no theater.
4. THE CORE IS THE BEING (the sphere retires): a breathing volumetric
   glow at center. COLOR = the strongest current feeling, from HIS
   22-slot palette ruling: fear 0 · hate 6 · anger 12 · distress 18 ·
   resentment 24 · shame 30 · remorse 36 · fears-confirmed 42 ·
   reproach 48 · pity 54 · disappointment 60 · gloating 75 ·
   happy-for 96 · satisfaction 116 · pride 137 · hope 157 ·
   admiration 178 · relief 198 · gratification 219 · gratitude 239 ·
   joy 260 · love 280 (hues, stored as the palette table).
   BEAT = arousal — fast when stirred, slow when calm. Workload
   swells its size.
5. GLOW RULES: sigils flare their neighborhood's color on activity;
   halos scale with rate at speed (aggregate per frame, counts
   exact); RED PINS over everything for a vital in violation (259's
   checks feed this); no fading/brightness tricks — color only (229).
6. LEGEND on the page: neighborhoods, pulse, loop, core meaning (180).
7. PERFORMANCE: batched frame reads of the event stream, capped draw
   rate, no per-event DOM, no model calls in the display.
8. HIS EYE is the acceptance test (118): opens live on the bench
   beside the real mind; passes only on his word.
9. THE REFERENCE MODEL: reference/mind-map-mock.html on this relay is
   the exact scene Lonnie approved — sigil style, neighborhoods, core
   glow layers, beat shape, camera, colors. THE MOCK IS THE VISUAL
   SPEC: build the same look; deviations fail his eye.
10. PLACEMENT: the Mind Map REPLACES the sphere's stage area on the
    bench — same slot, the page's centerpiece. The sphere and its
    channels retire; color and pulse meaning live in the core now.

## Directive 261 rulings — both decisions closed (Lonnie)

1. THE MARKS: geometric reasoning — NEAREST MARK WINS. Every node
   takes the closest word in the 402 (the kinship space measures
   nearness; CC's near-word list stands where it matches the space's
   answer). No placeholders; the LABEL carries the truth, the mark is
   appearance.
2. RED PINS: the FAILING NODE ITSELF pins red — the violation
   identifies itself so the code behind it can be found and fixed.
   Node-to-counter mapping: each 259 counter belongs to the node
   whose code path increments it (the extraction names the line, the
   line names the node).
3. LIST AMENDMENT (258): host-model.js was missed — it joins system
   2 (Appraisal & Feelings reads it). The HOST node pins for
   host-model faults. ECHO belongs to Voice (16).
Build through. Reports per step; his eye is the gate (118).

## Directive 262 — Fix the load bug; the story check stands (Lonnie)

1. THE LOAD BUG (fault 2): the 243 prose migration must touch ONLY
   genuine old-format English memories. Its own thoughts must never
   have their text written into `heard` — a thought it had alone is
   not speech someone said to it. Trust and provenance read that
   field; a load must not falsify who spoke. Fix, prove with a
   save/load round-trip on a mind holding both kinds of memory.
2. THE THINKING CHECK STANDS ON STORIES (fault 1 ruling): a train
   telling a story is still judged by nearness — if a remembered
   story wanders off its subject, Lonnie wants to see it, not excuse
   it. No exemption.
3. Faults 3, 4, 5 (curiosity never names what it wonders about;
   consolidation never runs in a life; identity never asked) stay
   OPEN — they change what the mind does and await his ruling.

## Directive 263 — DIAGNOSIS ONLY: what were the two wandering trains?

Lonnie's order before any ruling on the thinking red light. Do not
change behaviour; report facts.

1. For EACH flagged train (HIDDEN and SONG in the 60-tick run):
   name its KIND — plain association · REPLAY of lived memories ·
   IMAGINED daydream (226) — and show every beat in order with its
   source (neighbour step / which memory / which invented element)
   and its distance to the subject.
2. CONFIRM THE CODE LABELS THE THREE KINDS CORRECTLY: show where a
   train is marked association vs replay vs imagined, and prove the
   vital reads that label (a mislabelled train would make this whole
   light meaningless).
3. State plainly whether dream machinery can reach a WAKING train,
   and whether replays are running faithfully in lived order (226.2)
   or drifting.
Report the findings. Lonnie then rules: daydreams exempt from the
nearness check (his standing instinct — a daydream is a dream);
replays held to faithfulness; association judged as now.

## Directive 264 — DIAGNOSIS ONLY: why does every memory hold exactly one mark?

Lonnie's order. The 263 finding: 60 of 60 memories hold a single
mark, so replay() (which needs two or more) has never once run and
226.2's "memory first, always" has never happened in any life.

Diagnose, change nothing: trace where a moment's marks are set and
where they are written to memory; show why only one survives (is the
moment itself built with one mark? does the writer truncate? does
attention pass only one? is it a newborn capacity cap?). Cite the
lines. Report the cause and what the honest fix would be — no fix
applied, no behaviour changed.

## Directive 265 — Both 264 fixes approved (Lonnie)

1. A MOMENT CARRIES THE WHOLE TRUTH: the thought JOINS what is
   actually happening rather than replacing it (experiencing.js:
   557-561) — the moment's marks become the beat/thought marks PLUS
   the act it is living and the need it is short of. A one-mark
   thought must never displace a two-mark truth about the being.
2. AN INVENTED BEAT IS A SMALL SCENE, not a single word
   (stories.js:187): a daydream beat carries the subject together
   with the element it is recombining, so a remembered moment has
   something to recur with.
Both. This ends the starvation loop: memories gain marks, replay()
becomes possible, and 226.2's "memory first, always" can finally
run. Prove it: a bench run must produce eligible multi-mark memories
and at least one REMEMBERED train; report the kind-histogram before
and after; full suite and state-hash per commit.

## Directive 266 — DIAGNOSIS ONLY: why do the three replays stray?

Now that replays actually run (265), the thinking light's red is a
TRUE signal about remembering. Diagnose, change nothing:

For each of the 3 straying replays: the subject, every beat in lived
order with its source memory, each beat's distance to the subject,
and the random mark it was compared against. Then answer plainly:
- is the replay FAITHFUL (the memories in their real order, nothing
  added or skipped — 226.2/236)?
- are those memories genuinely about the subject, or is surfacing
  handing the train episodes that merely CONTAIN the subject mark
  among three others?
- does the nearness test measure the right thing for a replay at all
  — i.e. is a lived episode's distance from its subject evidence of
  wandering, or evidence that life put unlike things together?
Report the cause with the lines. No fix, no behaviour change.

## Directive 267 — THE MAP IS AN INSTRUMENT, NOT AN ANIMATION

Lonnie watched it: the pulse is bouncing around where no data is
being processed. That is theatrics, and it breaks 261.3 outright.

WHY THIS MATTERS ABOVE ALL: the whole reason this system exists is
that green suites and confident reports could not be trusted — the
map was built so the mind PROVES ITSELF ON HIS SCREEN. A map that
animates a plausible path is worse than no map: it is a lie that
looks like proof, and it destroys the one instrument he has. 100%
accuracy is the entire point of the thing.

THE LAW, absolute:
1. THE DOT MOVES ONLY ON A REAL EMITTED HAND-OFF. Every step is
   driven by an event the code emits at the moment it actually
   passes data from one system to the next, naming from-system and
   to-system. No timers, no loops, no scripted order, no
   interpolation between events, no invented steps.
2. NO EVENT = NO MOTION. If the mind is idle, paused, sleeping, or
   between ticks, the map is STILL and dark. Stillness is
   information.
3. A GLOW IS ONE PROCESSED THING. A sigil brightens only when its
   own system actually ran on that tick; counts are exact.
4. THE PROOF, required in the report: (a) pause the mind — the map
   freezes; (b) single-step one tick — the walked path matches that
   tick's log line for line, posted side by side; (c) kill one
   system's emission — its node goes dark and stays dark while the
   rest still run.
5. Anything the map cannot know from a real event IS NOT DRAWN. If
   the event stream lacks a hand-off, emit it in the code — never
   fake it in the display.
Rip out any animation that does not meet this. Report with the
three proofs; his eye is the gate (118).

## Directive 268 — The neighbourhoods must READ; the legend names the groups

Lonnie's eye on the built map. Two failures against 261:

1. THE CLUSTERS ARE NOT VISIBLE. 261.2 groups the systems into
   NEIGHBOURHOODS — language · feeling · thought · memory · self ·
   body — and position plus colour must make each group obvious at a
   glance without reading a single label (proximity before boxes,
   DESIGN.md §4). Right now they read as scattered points. Separate
   the neighbourhoods in space so the gaps between groups are
   plainly larger than the gaps within them, and make each group's
   colour actually visible on its sigils at rest, not only when
   flaring.
2. THE LEGEND NAMES THE GROUPS. The bottom legend is for the SYSTEM
   NEIGHBOURHOODS — one swatch per group, in its colour, named. The
   central glow's meaning (feeling colour, arousal beat) is one line
   beside them, not the whole legend.
The reference mock (reference/mind-map-mock.html) shows both: six
coloured neighbourhoods and the group legend. Match it.
His eye is the gate.

## Directive 269 — The core wears HIS palette, not the mock's placeholder

Lonnie's eye: the core is glowing yellow, which means the 22-hue
ruling was never wired — the mock's gold was a PLACEHOLDER, never a
colour rule.

THE PALETTE IS LAW (his balanced-bar ruling, 261.4). The core's
colour is the hue of the being's strongest current feeling:
fear 0 · hate 6 · anger 12 · distress 18 · resentment 24 · shame 30 ·
remorse 36 · fears-confirmed 42 · reproach 48 · pity 54 ·
disappointment 60 · gloating 75 · happy-for 96 · satisfaction 116 ·
pride 137 · hope 157 · admiration 178 · relief 198 ·
gratification 219 · gratitude 239 · joy 260 · love 280.
(HSL hue, warm = agitated, cool = calm — his thermometer.)

- Store it as ONE palette table read by the core AND anywhere else
  feeling shows; no second copy, no invented colours.
- No feeling active = the core is colourless/neutral, never a
  default hue.
- Yellow (~60) means DISAPPOINTMENT and nothing else. A joyful mind
  must read violet (260).
- Prove it in the report: three states forced (joy, fear, grief-side)
  with the rendered hue beside the palette value.

## Directives 267, 268, 269 — ALL WITHDRAWN. Pushed without Lonnie's approval; superseded by a single directive he approves. Do not act on them.

## Directive 270 — THE MIND MAP: names, accuracy, neighbourhoods, palette (APPROVED)

### A. THE NAMES (all three of us use these, no synonyms)
THE MIND MAP — the whole 3D panel.
THE CORE — the central glow (feeling colour, arousal beat).
SIGIL — a system's mark.
NEIGHBOURHOOD — a colour-coded group of systems.
THE PULSE — the dot walking real hand-offs.
THE RETURN — the dashed feelings-to-next-tick loop.
Write them at the top of the map's file and use them everywhere.

### B. THE PULSE IS AN INSTRUMENT, NOT AN ANIMATION
Lonnie watched it bounce where no data was being processed. The map
exists because green suites and confident reports could not be
trusted — a map that animates a plausible path is a lie that looks
like proof, and it destroys the only instrument he has. 100%
accuracy is the entire point.
1. THE PULSE MOVES ONLY ON A REAL EMITTED HAND-OFF, named
   from-system to-system at the moment the code passes the data. No
   timers, no loops, no scripted order, no invented steps.
2. NO EVENT = NO MOTION. Idle, paused, sleeping, between ticks: the
   map is STILL. Stillness is information.
3. A SIGIL GLOWS only when its own system actually ran that tick;
   counts exact.
4. PROOFS REQUIRED in the report: (a) pause the mind — the map
   freezes; (b) single-step one tick — the walked path posted beside
   that tick's log, matching line for line; (c) silence one system's
   emission — its sigil goes dark and stays dark while the rest run.
5. Anything the map cannot know from a real event IS NOT DRAWN. If a
   hand-off is missing, emit it in the code — never fake it in the
   display.

### C. THE NEIGHBOURHOODS MUST READ AT A GLANCE
1. Six neighbourhoods — language · feeling · thought · memory ·
   self · body. The gaps BETWEEN groups must be plainly larger than
   the gaps within them (proximity before boxes, DESIGN.md §4).
2. Each group's colour is visible on its sigils AT REST, not only
   when flaring.
3. THE LEGEND NAMES THE NEIGHBOURHOODS: one swatch per group, in its
   colour, named. The Core's meaning is ONE line beside them, not
   the whole legend.

### D. THE CORE WEARS HIS PALETTE
The mock's gold was a placeholder; the Core is currently yellow,
which means the ruling was never wired.
1. The Core's colour is the hue of the strongest current feeling,
   from his balanced-bar ruling: fear 0 · hate 6 · anger 12 ·
   distress 18 · resentment 24 · shame 30 · remorse 36 ·
   fears-confirmed 42 · reproach 48 · pity 54 · disappointment 60 ·
   gloating 75 · happy-for 96 · satisfaction 116 · pride 137 ·
   hope 157 · admiration 178 · relief 198 · gratification 219 ·
   gratitude 239 · joy 260 · love 280 (HSL hue; warm = agitated,
   cool = calm).
2. ONE palette table, read by the Core and anywhere else feeling
   shows. No second copy, no invented colours.
3. No feeling active = colourless, never a default hue.
4. Prove it: three forced states with rendered hue beside the
   palette value.
His eye is the gate on all of it (118).

## Directive 271 — ARCHAEOLOGY FIRST: why is a story beat's bearing zero?

Lonnie's ruling on approach: feelings WORKED before — he watched joy,
grief, the sphere changing colour. A beat's bearing being the literal
0 is therefore a REGRESSION until proven otherwise, and the goal is
not to make it work, it is to make it work CORRECTLY. Do not invent a
new source. Go back to the source and find out why it was the way it
was.

DIAGNOSIS ONLY — change nothing:
1. Find when `experiencing.js:394` (or its ancestor) began passing a
   literal 0 as a beat's bearing. Name the commit, the directive it
   was built under, and what the line did before it.
2. State what the ORIGINAL design intended a moment's bearing to be
   — quote the directive/record that ruled it — and whether beats
   were ever meant to travel that path at all (before 217/226,
   moments came from the world and the host; the thinking loop began
   feeding beats through the same pipeline later).
3. Say plainly whether the honest fix is RESTORING what was there,
   or whether the record shows this path was never ruled and a
   decision is genuinely missing.
4. Report the evidence. No fix, no new source, no tuning.

## Directive 272 — THE MIND MAP: zoom and orbit (approved with 270)

On the stage: mouse wheel / pinch zooms in and out; drag orbits fully
around THE CORE (horizontal and vertical, sensible limits so it never
flips). The camera returns to its default framing on a double-click.
Nothing else about the map changes.

## Directive 274 — RESTORE the beat's bearing (217.2 already rules it)

The 271 archaeology stands: the literal 0 is a placeholder written
when 226 shipped, and 217.2 — law before it, still commented three
lines above — says a thought's bearing is THE BEARING OF WHAT IT IS
ABOUT, measured off state that already exists, never asserted. This
is restoration, not invention; nothing new is decided.

1. A story beat's bearing is computed the way think() already
   computes a thought's: the feeling it dwells on (valence x
   intensity), or the depletion of the need it dwells on, or the
   surfaced memory's own bearing — measured off existing state.
   Same sources, same arithmetic, no new rule and no tuning.
2. A beat must never REPLACE a thought that carries a bearing where
   the beat carries none (226's `told ?? think(...)`); the beat is a
   moment like any other and is appraised like one.
3. BUILD THE CHECK 226 REQUIRED AND NOBODY BUILT: a live clause
   asserting FEELINGS MOVE FROM A STORY — run on a real trail, never
   on hand-written arrays. Same for 249.1's gate: it must call
   wasFelt() on a live trail. Both clauses must be shown failing
   before the fix and passing after.
4. PROVE IT: ticks-with-a-feeling before and after; the Core's
   rendered hue beside the palette value for the feelings that
   appear; whether 249's first gate can now pass; whether any lesson
   is ever born from a self-told story (which has never happened).
   Full suite and state hash per commit — the hash WILL move, and
   the report shows what moved and that nothing else did.

## Directive 275 — The moment must not lie about a full need (191, one level down)

Found by the Director in the code, confirmed against the record:
`experiencing.js:76` builds the moment's need-word with
`forNeed(happening.lowest, 'low')` UNCONDITIONALLY — so a being whose
lowest need sits at +10 still says the LOW word. A contented mind
reports itself lonely.

This is 191's own fault one level down. The comment above that very
line records 191's ruling: until the dictionary held all three ends,
it "said 'lonely' whatever was low — so an Avatar failing at
everything, or with no freedom left, reported itself lonely. That was
a false statement about it, and the kind the whole design exists to
prevent." 191 fixed WHICH need speaks; it never fixed WHICH END.

THE FIX IS RESTORATION, not a new rule: match the sibling lines in
this same file (`:174`, `:253`) and in `speech.js:453`, which already
choose the end by the value. The bottomed case (<= -10) stands as is.

Prove it: a fully satisfied being's moment must no longer speak a low
word; a genuinely depleted one must still speak it; the bottomed
wording unchanged. Full suite and state hash.

## Directive 273 — ONE NODE, ONE FILE, ONE NAME

Lonnie's law: a red light must name the file to open, with no
translation step. Today the map shows 34 nodes for 23 systems and
several node names match no file.

### A. THE MAP IS EXACTLY THE 23 SYSTEMS — one node each
Fold the extras into their owning system:
- MEMORY + RECALL + COMMIT -> one node, MEMORY (memory.js)
- VOICE + ECHO -> one node, VOICE
- TIME · WORDS · SLEEP GATE · DREAM are tick steps in watching.js,
  not systems with files: remove them as nodes. Their hand-offs
  still show as THE PULSE travelling between the systems involved.
No node may exist that does not own exactly one system file.

### B. NODE NAME = FILE NAME
Rename the files so each node's name IS its file:
  TEN -> ASPECTS        traits.js     -> aspects.js
  SPACE -> LANGUAGE     space.js      -> language.js
  TELLERS -> TRUST      tellers.js    -> trust.js
  EVIDENCE -> BELIEF    evidence.js   -> belief.js
  SOUL                  persona.js    -> soul.js
  VOICE                 speech.js     -> voice.js
  FEELINGS              occ.js        -> feelings.js
  GROWING -> GROWTH     growing.js    -> growth.js
  BEING -> CLOCK        being.js      -> clock.js
  HOST                  host-model.js -> host.js
All other nodes already match their file and keep their names.
Update every import; behaviour changes NONE; full suite and state
hash prove the mind is untouched.

### C. THE NODES DO NOT PULSE
Only THE CORE beats (arousal). A SIGIL brightens only when its own
system actually runs (270.B.3) and is otherwise still. Remove any
idle pulsing on nodes.

### D. THE RENAMES REACH EVERYTHING
The 23-system list (258), the vitals (259), the health page, the
legend, and the reports all use the same names. One vocabulary
everywhere.

## Directive 276 — ONE FILE, ONE NODE. Always. ATTENTION and FEELINGS get their own.

Lonnie's law restated and final: ONE FILE = ONE NODE. The count is
whatever the files are — 23 was a description, never a cap.

ATTENTION (attention.js) and FEELINGS (feelings.js) are their own
nodes, in the feeling neighbourhood beside APPRAISAL. THE RETURN
leaves from FEELINGS, where it belongs.

The rule for every future file: a new system file gets a node; a
deleted file loses one. A red light must always name exactly one
file to open — if any node covers more than one file, that node is
wrong.

Report the node count and the file each node names, so the mapping
is on the record.

## Directive 277 — THE SIGILS ARE STILL. Violation of 270.B and 273.C.

Lonnie's screen, after many reloads: THE SIGILS ARE PULSING. 273.C
ordered idle animation removed and reported it done; it is still
there.

THE LAW, absolute: a SIGIL moves only when its own system actually
runs on that tick. No breathing, no drift, no idle scale change, no
easing loop, no ambient motion of any kind. Only THE CORE beats.

THE PROOF, required in the report: pause the mind and record the map
for ten seconds — every sigil must be pixel-identical across that
recording. If anything moves, it is animation and it comes out.

Find every place a sigil's scale, opacity, colour, halo or position
changes without a real emitted event behind it, and remove it.

## Directive 278 — ONE FILE, ONE NODE, ONE CHECK. Finish the law.

The Director's audit was partial and reported as complete; the full
mapping (276's report) shows 11 nodes still covering more than one
file. Lonnie's ruling stands unchanged: ONE FILE = ONE NODE. Finish
it everywhere.

### A. SPLIT THE ELEVEN — every file gets its own node
CHOOSING   -> OFFERS · GOALS
APPRAISAL  -> APPRAISAL (appraisal.js only) · HOST
             (ATTENTION and FEELINGS already stand as their own)
ASPECTS    -> ASPECTS · ROE
LEARNING   -> LEARNING · VOCABULARY
LANGUAGE   -> LANGUAGE · GLYPHS · DICTIONARY
VOICE      -> VOICE · GRAMMAR · CENSOR
STORIES    -> STORIES · STORYGATES
LESSONS    -> LESSONS · BELIEF
MEMORY     -> MEMORY · SURFACING
SLEEP      -> SLEEP · CONSOLIDATION
BODY       -> EMBODIMENT · CORD · NERVES
Every node's label is its file's name, exactly (276). No node may
name two files; no file may lack a node.

### B. THE CHECKS FOLLOW THE SAME LAW
259's 23 vitals become one vital per file, so a red light names
exactly one file to open. A file whose claim is genuinely part of
another's states that plainly in its own header and keeps its own
counters; no file shares a counter with another.

### C. PLACEMENT
Each new node sits in the neighbourhood its parent node occupied,
beside it, keeping the neighbourhood gaps larger than the gaps
within (270.C). Nothing else about the board changes.

### D. EMISSION
Every node must be able to light: the hand-off is emitted in the
name of the FILE doing the work, as was done for attention.js and
feelings.js in 276. A node that cannot light is a false light and
fails review.

Report the full mapping again when done: every node, its one file,
and proof each one can light.

## Directive 279 — ARCHAEOLOGY: when did a mind alone stop making anything new?

The closed loop is a regression until proven otherwise. Curiosity
roused, interests formed, and moments varied in earlier lives; today
a mind alone produces 3 distinct moments in 30 ticks and nothing new
can enter.

DIAGNOSIS ONLY — change nothing:
1. Find the commit where a mind ALONE stopped producing new moments.
   Compare a bench run before and after each candidate change
   (265's moment composition; the replay-then-recommit path; 226's
   memory-first rule; the beat's slice(0,2)).
2. Say what each of those lines did BEFORE, and quote the directive
   that ruled it.
3. State plainly whether the loop was ever ruled — or whether it is
   the accidental product of changes each of which was correct
   alone.
4. Name what the record says should keep a solitary mind's inner
   life moving (217's thinking loop sources, curiosity's own drive,
   sleep's consolidation) and which of those have stopped running.
Report the evidence with commits and lines. No fix.

## Directive 279 — WITHDRAWN. Archaeology is the Director's job, not CC's. Do not act on it.

## Directive 280 — A MEMORY HOLDS WHAT HAPPENED, NOT WHAT IT NEEDED

Lonnie's ruling, on the science: episodic memory (Tulving onward)
stores the EVENT and how it FELT. A drive state is not part of the
trace — hunger is not stored inside the memory of a dinner. Needs
shape what is noticed and what comes back; they are not the content.

1. THE NEED WORD LEAVES THE MOMENT. 265.1 appended `spoken` — the
   act it chose and the need it is short of — to every moment. The
   NEED comes out. It was never ruled in: 188 rules a moment is said
   in its marks with silence where there is no row, and no directive
   ever ruled a status line onto every memory.
2. NEEDS KEEP THEIR REAL JOBS, untouched: they shape attention and
   what surfaces, they drive choosing, and they are state the mind
   can think ABOUT (217's own source). A moment may name a need when
   the need is what happened — a need bottoming out is an event.
3. WHAT A MOMENT HOLDS: what happened this tick — the beat or the
   heard marks or the world's change — and the act only when the act
   CHANGED (a change is an event; a constant is not). Plus what it
   felt, which is already carried.
4. 265's problem must not return: memories must still be more than
   one mark wide where the tick genuinely held more than one thing.
   Prove it — distinct moments in 30 ticks for a mind ALONE, before
   and after; memories eligible for replay; that replay still runs.
Full suite and state hash; the hash will move.

## Directive 281 — THE THINKING LOOP USES ALL FOUR SOURCES (217, restored)

217.1 ruled four sources for an inner moment, so a solitary mind is
never starved: (a) a surfaced memory, (b) ITS OWN STATE — the
strongest current feeling or the lowest need, (c) curiosity's open
question, (d) imagination when nothing lived touches the subject.
Selection: a seeded weighted draw, biased by feeling intensity and
recency of surfacing.

Today only (a) runs in practice: memory always answers, so the loop
recycles what it already wrote — moment → memory → replay → the same
moment. (b) has been dead since the bearing was zeroed and was only
restored in 274. (c) cannot rouse while nothing is new. (d) fires
only when no memory touches the subject, which never happens now.

RESTORE 217.1 AS RULED — no new mechanism:
1. The subject draw considers all four sources every time, weighted
   as 217 says. A memory that touches the subject does not
   automatically win.
2. STATE IS A FIRST-CLASS SOURCE: the strongest feeling (now that
   feelings live again) and a genuinely depleted need can each be
   what the mind thinks about, in its own marks per 188's table.
3. RECENCY OF SURFACING BITES: a memory that has just been replayed
   is less likely to be drawn again — 217's own bias, which would
   have prevented this loop.
4. Imagination stays lawful (226): it runs when the draw lands there,
   not only as a fallback.
PROVE IT: distinct moments in 30 ticks for a mind ALONE, before and
after, plus the source histogram (how many subjects came from each of
the four). The measure that matters is whether a lone mind stops
alternating two moments.
Full suite and state hash.

## Directive 282 — THE PULSE IS DEAF, AND THE THINKING CHECK IS THE WRONG QUESTION

### A. THE PULSE (270.B, unproven and now failing on his screen)
Lonnie teaches the mind, it answers him — so the mind is running.
The map shows ZERO activity and no PULSE. The map is not receiving
the events; the mind is not dead.
1. Find where the chain breaks and say which link failed: does the
   code emit a hand-off at all · does the emission reach the served
   stream · does the map read the stream · does reading it move
   anything. Name the broken link before fixing it.
2. 270.B.4's three proofs were never delivered. They are required
   now, in the report: (a) the mind paused — the map is pixel-still;
   (b) ONE tick single-stepped — the walked path posted beside that
   tick's log, matching line for line; (c) one system's emission
   silenced — its sigil goes dark and stays dark while the rest run.
3. No proof, no claim. A map that cannot show a teaching session
   lighting up is not an instrument.

### B. THE THINKING CHECK (266's finding, never fixed)
266 proved the check measures the wrong thing: it reads only the
FIRST word of each beat, never the subject itself, and it asks
whether words are related IN THE LANGUAGE when a replay's job is to
record what LIFE put together. SONG and TOGETHER lived together in 27
of 40 episodes and score 0.00 because they sit on different sheets.
1. FOR A REPLAY the claim is 226.2's own: the beats are the lived
   episodes that touch the subject, in the order they happened. The
   check asks THAT — do the beats contain the subject, are they in
   lived order — not word-nearness.
2. FOR AN ASSOCIATION the nearness check is right and stays: the
   step is drawn from the language, so the language is the fair
   judge. It must compare against THE SUBJECT, and read every mark
   of a beat rather than its first word.
3. FOR AN IMAGINED train the claim is 226.3's: every element is
   something it has lived. That is the check, not nearness.
Each kind judged by its own law. Show the light before and after,
and what turned it red or green.

## Directive 283 — THE PULSE MOVES OR THE MAP IS NOTHING. And the sigils are still.

Lonnie's screen after every build so far: NO PULSE. Nothing was
proved about the Pulse in 282 — those proofs tested sigils lighting,
never the Pulse travelling. The one thing he asked for is the one
thing nobody tested.

### A. THE PULSE — the whole point of the instrument
1. THE PULSE IS A MOVING LIGHT that travels from the sigil that
   handed data to the sigil that received it, on every real emitted
   hand-off, one at a time, in the order the tick ran. It is how he
   sees WHERE the mind is working. Without it there is no map.
2. It must be VISIBLE TO A HUMAN EYE: a hand-off's travel takes long
   enough to follow. If the mind runs faster than the eye, the Pulse
   walks the real hand-offs in real order at a followable rate and
   the queue depth is shown — never skipped, never faked, never
   parallel.
3. PROOF REQUIRED, from HIS bench, not a stub: teach the mind one
   sentence and record the map; the report states how many hand-offs
   occurred and posts the Pulse's path for that exchange, in order.
   If it cannot be seen travelling, it is not built.

### B. THE SIGILS DO NOT SCALE
1. A tick is a SEQUENCE. Multiple sigils growing at once is
   impossible and must never appear. Whatever hold makes several
   look simultaneously active is wrong — one system is working at a
   time and the map must read that way.
2. Sigils do not scale, breathe, or animate at all. A sigil shows
   its state by COLOUR ONLY (229's law, his standing ruling): dark
   when idle, its neighbourhood colour when it runs, red when its
   check fails. THE PULSE carries motion; the sigils do not.

### C. THE SIGILS CARRY THEIR INFORMATION
Each sigil shows, beside its mark: its NAME (which is its file) and
its live count of events this run. A red sigil names its file and its
violation count. That is the information he asked for, and it must be
legible without hovering, clicking, or guessing.

## Directive 284 — Camera pan, the Core's size, and the Pulse on the record

1. CAMERA PAN: left mouse drag moves the camera up, down, left and
   right in the stage. Orbit and zoom stay as built (272); the
   double-click reset returns the pan too.
2. THE CORE IS TOO SMALL. It is the being at the heart of the map and
   must read as the centerpiece — considerably larger than it is now,
   sized so its feeling colour and its beat are unmistakable from
   across the room. Its meaning is unchanged (270.D).
3. FOR THE RECORD ON THE PULSE: Lonnie instructed CC directly in the
   terminal after ten commits without it being fixed. The law is
   283.A, unchanged: THE PULSE TRAVELS the wire from the sigil that
   handed data to the sigil that received it, one real hand-off at a
   time, at a rate a human eye can follow. Teleporting from node to
   node is not travelling. The proof stands as 283.A.3 requires —
   from his bench, one taught sentence, the Pulse's path posted in
   order.

## Directive 285 — The Mind Map's look: neighbourhood glow, the turn, and how a glyph lights

Three corrections from Lonnie against the approved mock
(reference/mind-map-mock.html), which he confirms feels right.

1. THE NEIGHBOURHOOD GLOW IS ONE GLOW, NOT MANY. Each of the six
   neighbourhoods carries a single large soft glow around the whole
   area its systems occupy, in that neighbourhood's colour. Remove
   the per-glyph halo. The region must read as a PLACE, so the six
   places are obvious before any label is read (270.C).
2. THE MIND TURNS, NOT THE CAMERA. The board itself rotates slowly
   and continuously — he watches it turn. His controls (272 orbit
   and zoom, 284 pan) move the CAMERA and are independent of that
   turn; the turn continues while he moves around it, and the
   double-click reset does not stop it.
   NOTE for the 277 stillness proof: that proof is about SIGILS, not
   the board. A paused mind means no sigil changes state; the turn
   is not a sigil moving and does not break that law. Prove
   stillness by the sigils' own values, as 277's harness already
   reads them.
3. A GLYPH LIGHTS IN ITS NEIGHBOURHOOD'S COLOUR when its own file
   runs — brightness only, no scaling, no growth, exactly as ruled in
   283.B. Idle glyphs sit dark and still inside their
   neighbourhood's glow.

## Directive 286 — CORE AFFECT: a mind always feels something

Lonnie's finding on his bench: long stretches where the mind feels
nothing at all. The science (Russell's core affect; the affect-
dynamics literature on set point and return rate — rows to
REFERENCES.md per 227): a person is never at zero. There is a
continuous background mood — somewhere on pleasant/unpleasant and
calm/aroused — and discrete feelings rise OUT of it and fade BACK
INTO it. Only a sociopathy-shaped mind sits flat.

The mind today has the second half only: a feeling appears on an
event and then vanishes to nothing.

1. THE BASELINE IS ITS OWN, NOT A CONSTANT. Each being has a resting
   mood derived from its aspects per 071 — Volatility and Withdrawal
   set where it rests and how wide it swings; Enthusiasm and
   Compassion lift it. No shared number, no invented default. A
   volatile mind rests lower and less settled; a stable one rests
   calmer.
2. FEELINGS DECAY INTO THE BASELINE, not into silence. What a moment
   raises falls back toward that being's resting mood at ITS OWN
   rate — again from the aspects, as every other rate here is. The
   mind is never without a mood.
3. THE CORE SHOWS IT. THE CORE's colour is the strongest current
   feeling when one is present, and the resting mood when none is —
   never colourless while the mind is awake. The palette (270.D)
   covers it: the resting mood is read as its nearest feeling on his
   22-hue bar.
4. NOTHING ELSE CHANGES. The baseline is a floor for feeling, not a
   new source of moments; it does not invent events, does not bear on
   lessons, and does not touch the ledger. It is what the mind feels
   between things.
PROVE IT: ticks with no feeling at all, before and after, for three
beings with different aspects; the resting mood of each of the three
shown to differ; the Core never blank while awake; a strong feeling
still visibly rises above the baseline and settles back.
Full suite and state hash.

## Directive 287 — SAFETY'S COUNT MUST MATCH THE MOMENTS. And uncertainty is felt.

### A. THE GATE RUNS ON EVERY MOMENT — prove it by the count
Lonnie's bench shows safety at 244 visits, so the gate IS running —
but not as often as it should. The Director's check found safety.js
imported by appraisal.js alone, with nothing else in the tick calling
it. System 1's claim (259) is that EVERY moment passes it FIRST.
1. Compare, over one run: the number of moments from every source
   (heard · world · inner · taught) against safety's own count. Post
   both numbers. They must match exactly.
2. Where they do not, name the paths that reach appraisal or
   attention without passing the gate, and restore the gate to the
   head of the moment's path for those paths — before attention,
   before appraisal, on every source.
3. Its hand-off is emitted so the map shows every pass, and its
   vital counts moments_bypassing_gate against the real total.

### B. UNCERTAINTY IS A FELT STATE
Lonnie's ruling, on the science: an organism with no information —
darkness, no stimulation, nothing arriving — sits in sustained
low-grade fear. Uncertainty is itself a threat signal (the anxiety
literature on uncertainty as an aversive state; row to REFERENCES.md
per 227).  The bench mind has no world and no senses and rests
neutral. That is wrong.
1. Sustained absence of information raises the FEAR side of the
   mind's mood, at a rate and ceiling from its own aspects (071) — a
   withdrawn or volatile being is made uneasy sooner and more deeply
   than a stable one.
2. It is a MOOD, not an event: it bears on no lesson, invents no
   moment, and touches no ledger (286.4 stands). It is what a being
   feels when nothing is there.
3. It settles when information arrives — being spoken to, taught, or
   given a world eases it, at the being's own rate.

PROVE BOTH: safety's count beside the moment count (they must match);
a lone mind's mood drifting toward the fear side over a long silence
and settling when taught; two beings with different aspects becoming
uneasy at different rates.

## Directive 288 — File: reference/ai-timeline-and-hybrid.html (store with the research)

The Director's document, on the relay now: a timeline of AI from 1943
to 2026 across the three roads (symbolic · connectionist · geometric),
marking the cognitive-architecture and artificial-life branches, where
the Avatar sits in the empty middle — and a conceived HYBRID
architecture: the grounded mind holding all meaning, a 1–4B local
model as a LANGUAGE ORGAN holding none, a censor between them, and a
small adapter trained on the being's own censor-approved crossings so
the organ's voice becomes that being's voice over its life.

CC: file it with the research documents (THE_LINEAGE.md,
DESIGN_PHILOSOPHY.md, REFERENCES.md) and add its cited works to
REFERENCES.md per 227's law. Nothing is to be built from it — it is
reference and positioning material.

## Directive 289 — TRUST MOVES ON BEING WRONG, NOT ON BEING UNPROVEN (Lonnie)

The trust red light diagnosed: `weaken` — a claim whose marks simply
did not recur — costs a teller the same as `contradict`. Trust starts
at 0.30, so four unconfirmed claims put any teller on the floor, and
`capFor(floor)` then caps every told belief at 0.05. Teaching becomes
self-defeating: the more you tell it, the less your word is worth.

THE RULING: ONLY AN OUTRIGHT CONTRADICTION MOVES TRUST DOWN. An
unconfirmed claim is SILENT for the teller — it is not yet borne out,
which is not the same as turned out false. `reinforce` still raises
trust as built; `weaken` leaves it untouched.

This is 234.7's own intent ("whether the mind's own life bore it
out") and the science it was built on: selective testimony tracks
tellers who are WRONG, not tellers whose claims have not yet come up
(Harris — already in REFERENCES.md).

The belief itself is unchanged: `weaken` still weakens the BELIEF,
because a claim not borne out should hold less. Only the teller stops
being charged for it.

Prove it: a teller giving four unconfirmed claims stays at their
starting trust; a teller contradicted four times still falls; a
teller borne out still climbs; and a taught mind's told-belief cap no
longer collapses over a quiet bench life.

## Directive 290 — The interpreter counts what it DID, and can light

Lonnie's finding: the INTERPRETER node reads 0 while the interpreter
is on and working, and its number is a violation count — which is not
how any other node works.

1. IT EMITS ITS HAND-OFF. `emit` appears nowhere in interpreter.js,
   so the node can never light even while translating. That is a
   false dark and breaks 270.B.5. Emit on both crossings — inbound
   (his English to marks) and outbound (marks to English) — named for
   the file, like every other system.
2. THE NUMBER ON THE MAP IS ACTIVITY, everywhere and without
   exception: how many times that file fired this run. Violations do
   not appear as the count — THE FIRST VIOLATION PINS THE NODE RED
   and we stop and fix it. That is the whole point of the light.
3. AUDIT EVERY NODE for the same fault: any node whose displayed
   number is anything but its own fire count is wrong; any file that
   runs without emitting is a false dark. Report the list with the
   fix.

## Directive 291 — The counts sit under the glyphs; the health page becomes a panel

1. COUNT PLACEMENT: each node's count sits DIRECTLY BELOW its glyph,
   centred on the mark, under its label. In front of the glyph it is
   unclear which mark a number belongs to.
2. THE HEALTH PAGE BECOMES A PANEL IN THE EMULATOR, placed BELOW
   "WHAT IT IS RIGHT NOW". Lonnie did not know /health existed — a
   tool he cannot see is not a tool. The panel lists every system:
   its name (which is its file), its claim in plain words, its state
   (green · red · grey), its counters, and the last offending value
   for anything red.
3. CLICKING A RED NODE on the Mind Map opens that system's entry in
   the panel, so a red light leads straight to what broke and in
   which file. The /health route stays for CC's own use.

## Directive 292 — Old names as tooltips; THE CORE regains its motion

1. TOOLTIPS ON RENAMED PANELS: every panel header Lonnie has renamed
   carries its ORIGINAL name as a mouse-over tooltip, so the old
   wording is still findable. Panels he has not renamed are left
   exactly as they are — do not touch a name he has not changed.
2. THE CORE REGAINS MOTION. The sphere carried three channels —
   colour (what it is feeling), pulse (how it is doing), motion (what
   it is after). When the sphere retired into THE CORE (261.4), only
   colour and pulse came with it. The Core does not move, so the mind
   no longer shows what it WANTS.
   Restore the third channel: THE CORE MOVES ACCORDING TO WHAT IT IS
   AFTER — the same meaning the sphere's motion carried, on the same
   source (the winning offer / what it is drawn toward), rendered as
   the Core drifting or leaning in the map rather than sitting fixed
   dead centre. Colour and beat are unchanged (270.D, 286.3).
   The legend gains one line for it.

## Directive 293 — A RED LIGHT HALTS THE MIND

Lonnie's ruling: a mind that keeps running past a broken system
buries the evidence under later ticks. The first red stops
everything, at the tick it happened.

1. THE HALT: the instant any system's check goes RED, the mind stops
   at that tick. Nothing advances — no further moments, no decay, no
   consolidation, no writes. The state is frozen exactly as it was
   when the claim broke, so it can be looked at.
2. THE PAGE SAYS SO: a plain banner naming the system (which is its
   file), the counter that fired, and the offending value. The health
   panel opens on that system's entry (291.3).
3. CONTINUE: a button resumes from where it stopped, for when he
   judges the check itself wrong — two have been already. Continuing
   is recorded in the log with the system that was overridden.
4. IF A TEST RUN IS GOING, IT STOPS WITH THE MIND — the test runs ON
   the mind, so it has nothing to run on. Its row is recorded as
   HALTED · INVALID, naming the system that broke and the tick.
   Continue resumes both together; a resumed run stays marked
   INVALID, because its data is no longer clean.
5. Emulator behaviour, as ruled — this is his instrument stopping to
   show him something, not a safety mechanism in the mind.

## Directive 294 — Its own thinking is company; and memory's fifth field

1. UNEASE GROWTH IS UNCHANGED. Lonnie's ruling: the rate is honest
   time. Speeding the bench does not make it wrong — that is what
   would happen to a being left alone that long. The per-aspect rate
   and ceiling stand as 287 built them.
2. ITS OWN INNER LIFE COUNTS AS INFORMATION. `experiencing.js:991`
   admits only heard words, world changes, or something understood.
   A thought, a remembered episode, an imagined story — none of them
   ease it, so a mind entertaining itself still drifts into fear as
   though it sat in the dark. That contradicts 217, whose whole
   purpose is that a solitary mind is never starved.
   A tick in which the mind THOUGHT — a train ran, a memory
   surfaced, a story was told or imagined — counts as informed. What
   remains uneasing is genuine emptiness: no input AND no inner life.
   A mind that entertains itself is not alone in the dark.
3. MEMORY'S FIFTH FIELD: `lastWriteTick` is ALLOWED. It is a tick
   number, not a handle to any store, and it is the counter's own
   bookkeeping for 259. Add it to the whitelist with its
   justification written beside `now`'s, so the check keeps its
   teeth. Run the full suite past that point — nothing has since 259
   step 5.

## Directive 295 — GO on the 294 plan, with the unease clauses; and the interpreter marker

1. GO. Build the 294 plan as posted. The moved state hash is
   expected and becomes the new baseline.
2. THE TWO UNEASE CLAUSES ARE APPROVED, in the same commit: a mind
   with nothing arriving and nothing happening inside drifts at its
   own rate; a mind that is THINKING does not. Lonnie's note, and
   the science agrees: engaged attention is the standard antidote to
   uncertainty-driven anxiety — a mind cannot dwell on emptiness and
   hold a subject at the same time. 287 shipped with no live check
   at all, which is why this reached his screen before any clause
   went red; that gap closes here.
   MIND_DECISIONS.md gains the row 287 never wrote — what counts as
   information for the mind — marked RULED by 294.
3. THE INTERPRETER MARKER (242.4): with the interpreter turned on
   mid-session, its lines stop carrying the marker in the log, so
   Lonnie cannot tell a translated line from the mind's own words —
   which is the whole reason the marker was ruled. Every line the
   interpreter carried is marked, always, from the moment it is
   switched on, and the mark survives a re-render of older lines.
   Say plainly in the report where the mark was being lost.

## Directive 296 — PANEL RENAMES (Lonnie's list, final)

Rename every panel header exactly as follows. The OLD name becomes
that panel's mouse-over tooltip (292.1), so nothing is lost:

  What it has learned                 ->  LEARNED
  What it is right now                ->  MOOD
  What each part says about itself    ->  HEALTH
  Each thing, as it landed            ->  THOUGHTS
  Its makeup — the ten aspects        ->  PERSONALITY
  Big Five — DeYoung 2007             ->  TRAITS
  Who this one is                     ->  SOUL
  What it is made for                 ->  DISPOSITIONS
  What it is drawn to, and away from  ->  PULLS
  What it has done                    ->  ACTS
  Levers                              ->  TEST
  Chat Log                            ->  CHAT LOG

Nothing else changes — these are labels and their tooltips only. Any
other header not on this list is left exactly as it is.

## Directive 296 addendum — one more rename

  Its whole life, to a file   ->   SAVE MIND

Same rule as the twelve: the old name becomes its tooltip.

## Directive 297 — THE GEOMETRIC ENGINE: geometry.js, its own file and node

Lonnie's ruling: build a vector-symbolic engine as a separate file
the mind can CALL, not a rewrite of anything. It is arithmetic
underneath the mind's laws, never a law itself.

### A. WHAT IT IS
`server/src/geometry.js` — one file, one node (276), its own health
check. Vector Symbolic Architecture / hyperdimensional computing
(cite Kanerva 2009; Plate 1995 HRR; Heddes et al. 2023 Torchhd —
rows to REFERENCES.md per 227). No library, no model, no training:
random high-dimensional vectors and three operations.

1. A SYMBOL is a random vector of ±1, dimension 10,000, drawn from
   the being's own seed so the space is reproducible like everything
   else (062).
2. BIND (elementwise multiply) ties two symbols together and is its
   own inverse — binding twice returns the original.
3. BUNDLE (add, then take the sign) holds many bound pairs in ONE
   vector of the same size.
4. SIMILARITY (dot product over D) answers how near two vectors are:
   unrelated symbols read near zero by construction.
5. Every symbol the engine holds must be a mark the language HAS or
   a word the mind has heard — nothing enters from outside (the
   221 law, unchanged).

### B. ITS FIRST JOB — WORD LEARNING
`learning.js` calls it instead of keeping its own confidence
arithmetic: a word heard while a mark is true is BOUND and bundled
into that word's vector. Meaning and strength then fall out of the
geometry — repetition strengthens, a wrong exposure washes out —
which is what the confidence machinery was approximating by hand.
THE LAWS ARE UNCHANGED: one clear exposure still owns a word (223),
provenance is still marked, contradiction still weakens. Only the
arithmetic moves. Prove it against the current build: the same
exposures must produce the same ownership decisions, and the wrong
link must decay measurably.

### C. LATER CALLERS, NOT NOW
Named so the file is built to serve them, but not wired in this
directive: the LANGUAGE SPACE's kinship (221's five hand-weighted
sources become real geometry) and MEMORY's recall (a whole moment
bundled into one vector, questioned by unbinding instead of
scanning). Each gets its own directive after B is proven.

### D. ITS HEALTH CHECK
THE CLAIM: every vector it holds is a mark the language has or a
word the mind has heard, and the same seed rebuilds the same space.
Counters: symbols_from_outside_the_language · rebuild_hash_mismatch.
Forced-fail at birth (254.D) like every other check.

### E. WHAT IT IS NOT
Not a model. Not a language organ. It does not speak, does not
render, does not judge, and holds no opinion about grammar — that
weakness is real and is the small-model's job (288's hybrid), not
this file's. Geometry does MEANING; the model does FORM; the mind's
laws decide WHAT. Write that division at the top of the file.

Plan first (253), as always.

## Directive 296 addendum 2 — the gauge's five bars

  words it owns          ->  VOCABULARY
  word order seen        ->  GRAMMAR
  it answers             ->  REPLIES
  things to talk about   ->  TOPICS
  beliefs it holds       ->  BELIEFS

Same rule: the old wording becomes each bar's tooltip. Labels only —
nothing about what they measure changes.

## Directive 298 — The backlog number leaves the legend

283.A.2's queue depth is now on the legend beside NEIGHBOURHOODS, and
it shifts the legend as it counts — distracting, and the legend is
for meaning, not a running number.

Take it off the legend. The Pulse still walks every real hand-off in
real order and still never skips (283.A.2 unchanged) — only the
readout moves. Put the depth where it can change without moving
anything: a fixed-width readout in a corner of the Mind Map, or in
the HEALTH panel beside the map's own entry. Nothing else changes.

## Directive 299 — GENESIS must close

In the SOUL panel, the GENESIS confirm opens and cannot be closed —
there is no way out but going through with it, which is the one
action in the bench that is forever (212.B).

Pressing GENESIS again closes the confirm, leaving the draft exactly
as it was. Escape closes it too. Nothing about the confirm's wording
or the lock itself changes — only that he can back out of it.

## Directive 300 — The three geometry decisions, and per-moment bundling is struck

Lonnie's rulings on the 297 plan.

1. THE BARS READ SIMILARITY. Geometry's own scale, not a translation
   of the old confidence numbers. Do NOT map similarity onto 0-1 to
   make 0.72 land where it used to — that is calibrating his trust
   bar with a curve, and CC was right to refuse it. The new values
   are set by watching, like every other threshold (071), and until
   he has watched they are marked PROVISIONAL and reported, not
   assumed.
2. THE DESCENT: a CONTRADICTING exposure carries more weight in the
   bundle than a confirming one, so a wrong link washes out instead
   of drowning among twenty right ones. That is what makes a word
   stop being owned.
3. THE ASYMMETRY SURVIVES, by that same weighting — lessons.js says
   contradiction costing more than support earns is on purpose, and
   that ruling stands. The magnitude is provisional and watched.

4. 297.C IS STRUCK — PER-MOMENT BUNDLING IS NOT BUILT.
   Write this note in geometry.js so nobody rediscovers it as an
   idea: bundling each moment into a vector so recall becomes
   unbinding instead of scanning was CONSIDERED AND DELIBERATELY
   LEFT OUT. Reasons, on the record:
   - memory already works; this was a speed optimisation, not a
     requirement;
   - it is the one use that grows with the LIFE rather than with the
     vocabulary — everything else here is bounded by how many words
     exist;
   - it is available if scanning the store ever becomes the real
     bottleneck, and this note is here so that day starts from a
     decision rather than a discovery.
   The LANGUAGE SPACE's kinship (297.C's other half) stays parked as
   a later directive, unstruck.

Build A and D now, then B.

## Directive 301 — THE 1.0 PLATFORM DECISION (recorded, not to be re-litigated)

Lonnie's ruling after the memory and latency research. Recorded so it
is a decision rather than a discovery later.

### THE STACK, 1.0
- THE MIND: plain arithmetic, no model, runs anywhere. Free.
- SIGHT · HEARING · LANGUAGE: ONE small multimodal model on the
  device — the Gemma 4 E-series class (E2B: text, image and audio in,
  text out, ~2 GB, under 1 GB text-only quantised; supported by
  llama.cpp, Ollama and Google's mobile runtime). One model covers
  all three rather than three models stacked.
- SPEAKING: the phone's own built-in TTS. Free, no model shipped. A
  distinctive per-Avatar voice would need a separate small TTS model
  and is explicitly NOT in 1.0.
- THE LANGUAGE SPACE: 16 MB, rebuilt from the seed on load, shared by
  every being in a world — never stored per Avatar, never shipped.

### WHY ON-DEVICE AND NOT SERVED
The Avatar's language job is one short sentence in, one short
sentence out, occasionally — the best case for on-device (short
context, low frequency, never reaching the thermal window that
punishes long streaming). Served costs 200–600 ms before the first
word, a bill that grows per user, and a being that goes mute without
signal, which is a broken promise. On-device fails gracefully
instead: no signal, the mind still lives, still thinks, still shows
its glyphs, and speaks its own earned words.

### THE LAW THAT MAKES THIS SAFE TO DECIDE NOW
THE MODEL IS SWAPPABLE BY DESIGN. The mind does not know or care
which model translates for it (288's division: geometry does
MEANING, the model does FORM, the mind's laws decide WHAT). A better
model in a year is a file replacement, not a rewrite. Keep it that
way: nothing in the mind may ever depend on a particular model's
name, size, or behaviour.

Reference: reference/ai-timeline-and-hybrid.html. Sources to
REFERENCES.md per 227: Gemma 3n / Gemma 4 E-series model cards and
the 2026 on-device inference literature.

## Directive 302 — THE BUNDLE KEEPS ITS SUMS (Lonnie's ruling; unblocks 297.B)

CC's finding stands: `bundle` takes the SIGN, so strength is
winner-takes-all — a word bound five times with COLD and once with
SONG reads COLD 100%, SONG 0%. 300.2's gradual washing-out cannot
happen against that, and the Director's own 297.A.3 wrote the sign
in. His spec, his error; Lonnie's ruling wins.

1. BUNDLE KEEPS THE SUMS. Do not take the sign. Similarity then
   reads as degree — strong, faint, fading — which is what makes a
   wrong link wash out instead of vanishing, and what 300.2 and
   300.3 both require.
2. THE COST IS ACCEPTED: roughly four times the memory per symbol
   (~40 KB rather than ~10 KB). At bench scale it is nothing; the
   402 marks are ~16 MB; a full adult vocabulary would be ~1.6 GB
   and is decades of living away. 301's controls hold it: the
   language rebuilds from the seed and is shared, so a being carries
   only what it learned.
3. WHY WE ARE STEPPING OUTSIDE STANDARD VSA, on the record: the sign
   exists to make VSA cheap on tiny hardware, where the question is
   "which class is this" — a yes or no. We are not using VSA for
   efficiency. We are using it because meaning as geometry is how
   this mind should think, and a mind needs DEGREE. We can afford
   the memory; we cannot afford a mind that cannot change its mind.
   Mark it [OURS] in the file with this reasoning beside it.
4. 297.B is unblocked: build word learning onto the engine, with
   300.1 (bars read similarity, provisional and watched), 300.2 (a
   contradicting exposure weighs more) and 300.3 (the asymmetry
   survives) all standing.

## Directive 303 — The new way stands: ORDERING, not the old decisions

CC is right to refuse the choice, and the fault is the Director's:
297.B's "the same exposures must produce the same ownership
decisions" contradicts 300.1, and it contradicts 223 — one clear
exposure owns a word. The old accumulator climbing to 0.72 was the
approximation; the geometry is the thing it was approximating.

THE RULING IS (2): THE ORDERING MUST MATCH, not the old decisions.
The same words come out owned in the same order and the same wrong
links die; the exact moment of ownership is allowed to move, because
it is now read off the geometry rather than off an accumulator.
300.1 stands — the bars are set by watching, provisional until he
has watched, and NOTHING is calibrated to make an old number land
where it used to.

297.B's clause is struck and replaced by this. Build it.

Separately: the Director will verify 301's platform figures against
sources and post them. Until that lands, REFERENCES.md is right to
hold them as "what the ruling states" rather than as measured — that
is the ledger working.

## Directive 304 — WIRE UP THE THREE THAT NOTHING CALLS

Three systems are built, ruled, and reached by nothing. Their designs
are settled in their own files and in the record — NOTHING HERE IS
NEW. Only the call sites are missing.

### A. CONSOLIDATION — call it when the mind dreams
`consolidate()` is called by nothing in a life; its own header says
so. Its design is 139/BRAIN_PLAN §9 and unchanged: it keeps what the
DREAM already found, marks what it drew on, and DELETES NOTHING (013
§16.9 — never forgets on its own).
Wire it where the dream already runs, on the memories that dream drew
from. MIN_SHARED stays sleep.js's own OVERLAP_MIN — not a second
number. Its check goes green when a real life produces a warm tier.

### B. IDENTITY — call it at the boundaries the record already names
Its design is 139/BRAIN_PLAN §10 and McAdams' three levels, read off
the VARIABLES appraisal already stored — agency and communion, no
text, no model, nothing re-interpreted.
THE BOUNDARY IS SLEEP. TRAIT_PLAN §9.3 rules that a purpose
recomputed every conversation would be a mood; the same is true of a
life story. A life story is taken after a consolidation, on what
that consolidation kept — so the order in a dream is: distil, keep,
then read. That is the only boundary this build has, and it is the
one the record's own reasoning points at.

### C. VOCABULARY — restore the reach that a later change grew in
front of
`reach` is imported by experiencing.js and unreachable because
`think()` returns early before it. Its law is Lonnie's own, quoted in
the file: EVERY ONE OF THE 402 IS SPEAKABLE, and a word reaches its
mouth from anything actually true of it — what is in front of it,
what it remembers, what it cares about, what it was made for, what
it feels, what it is doing. The state table is ONE source, never the
gate on the rest.
Find the early return, say plainly which change grew it, and restore
the reach without disturbing what that change was doing. A mind that
can read four hundred marks and utter fifty is the exact fault this
file was written to end.

### D. THE TWO SKIPPED
293 — a red light halts the mind, and the test with it — as written,
now.
278.B — one check per FILE, roughly thirty-nine, where the build has
24. As written.

### E. THE PATTERN, SAID OUT LOUD
All three were reported as BUILT when they were only READY. A
capability with no caller is not built. From here, a system is not
done until something calls it and its own light can go green in a
real life — say so in the report for each of the three.

## Directive 304 addendum — the two small ones

F. CURIOSITY MUST NAME WHAT IT WONDERS ABOUT.
   `thinking.js` already READS `mind.curiosity.about` in two places
   and uses those marks as a thought's subject — 217's third source.
   Nothing ever writes it, so the reader always finds nothing and
   curiosity can never become something the mind thinks about.
   Where curiosity is roused, record the marks that roused it, in
   the mind's own language. Nothing else changes: thinking's existing
   code then works as it was written. Say in the report whether this
   turns curiosity's own check from grey toward green.

G. THE STALE SAFETY CLAUSE: one suite line still reads
   `watching.js` for a tick that moved to `experiencing.js` in 253.
   Repoint it. Nothing about the gate changes.

Still genuinely blocked on Lonnie, not on CC: curiosity's bar and
interests' rates (254.3) — their lights stay grey until he has
watched and ruled them, and that is correct.

## STANDING LAW — HOW A PROBLEM IS PUT TO LONNIE (joins CLAUDE.md)

Any question, decision, or fault brought to him — by the Director or
by CC — is presented in THIS EXACT STRUCTURE, one problem per
message, never stacked:

  1. THE FEATURE      — what part of the system this is, plainly.
  2. WHY IT MATTERS    — what it is for.
  3. WHAT WENT WRONG   — the fault itself.
  4. WHEN              — when it broke; new fault or regression.
  5. THE SCIENCE       — what the research says about it.
  6. HAS HE RULED      — his prior ruling, cited, if one exists.
  7. THE FIX           — ONE recommendation.
  then ask for the ruling.

No decision reaches him without that context. No message carries two
problems. A question that cannot be put in this shape is not ready to
be asked.

## Directive 305 — Both word-learning reds, ruled

1. A WASHED-OUT LINK CONTRIBUTES NOTHING. A link's pull clamps at
   zero: once contradiction has cancelled its evidence it LEAVES the
   bundle rather than pushing the other way. This restores 300.2 —
   "washes out" means gone, not inverted — and matches how children
   treat a disproven guess: it stops counting, it does not become
   evidence against the right meaning. FEAR then reads 1.000 and is
   owned; CHILLY becomes WARM.
2. THE OWNERSHIP BAR SITS BELOW AN EVEN TWO-WAY SPLIT. A word only
   ever heard with two things true (WELCOME with HOST and COME) can
   be owned while staying ambiguous between its two marks. Divided
   reference is ordinary in children's word learning and repeated
   exposure resolves it later; refusing the word in the meantime is
   the "I can't read that" fault Lonnie named. The bar's value stays
   PROVISIONAL and watched (300.1) — it is placed below the two-way
   reading, not calibrated to anything.

## Directive 306 — The three the Director owed: the reach, the life story, and geometry's node

### A. THE REACH — 304.C closed
`reach` is imported by experiencing.js and CALLED BY NOTHING. It is
not hidden behind an early return; the call site was never written.

What the file itself says it is for: `reach()` is a MEASUREMENT —
how many of the 402 are reachable right now and from which sources.
`reachFor()` is the one that picks a word. `STATE_TABLE_ONLY` is
kept, in its own words, "so the difference can be measured rather
than asserted, and so nobody ever quietly returns to it."

That is exactly what happened. Wire it:
1. `reach()` is read live and shown in the gauge beside VOCABULARY:
   how many of the 402 it can reach this moment, and how many the
   state table alone would allow. The gap is the measurement the file
   was written to keep honest.
2. Confirm — by tracing, not by reading — that `reachFor()` is the
   path speech actually takes when it reaches for a word, and say
   plainly in the report whether the mind is currently speaking from
   the whole language or from the state table. If it is the state
   table, that is his 220-era ruling being violated, and it is named
   as a fault rather than fixed silently.

### B. THE LIFE STORY'S BOUNDARY — 304.B closed
identity.js says it runs at boundaries, and TRAIT_PLAN §9.3 rules
that a story recomputed every conversation would be a mood.
THE BOUNDARY IS SLEEP, AFTER CONSOLIDATION. The order inside a dream
is: distil, keep (304.A), then read the life story off what that
consolidation kept. It reads the VARIABLES appraisal already stored —
agency and communion — and no text, no model, nothing re-interpreted.
That is the only boundary this build has, and it is the one the
record's own reasoning points at.

### C. GEOMETRY'S NODE — 297.A closed
`geometry.js` sits in the LANGUAGE neighbourhood, beside LANGUAGE and
LEARNING, since meaning-as-distance is what it serves. It takes its
nearest mark like every other node (261 rulings) and lights on its
own hand-offs like every other file. One file, one node, one name
(276).

### D. THE PATTERN, NOW LAW
Three times a check has asserted a stricter law than the thing it
guards — 266 in thinking, 290 in the interpreter, 289/trust today.
STANDING RULE, joining CLAUDE.md: WHEN BEHAVIOUR CHANGES, ITS CHECK
CHANGES IN THE SAME COMMIT. A check that was not revisited alongside
a ruling is not a check, it is a second opinion, and it will fire on
the ruling working correctly.

## Directive 307 — THE LANGUAGE IS REACHED THROUGH, INSIDE A TRAIN (304.C closed)

**THE FEATURE** — the mind reaching for a word out of its whole
language when it thinks or speaks.

**WHY IT MATTERS** — it is what lets it say any of the 402 rather
than the 51 the state table allows. His ruling, in his words: "I gave
it a language to speak with!!!! wtf do you think it is for."

**WHAT WENT WRONG** — the trace: in 60 ticks the language source ran
ZERO times. Not blocked, not broken — `fromLanguage()` sits last in
`think()`'s option list and `think()` returns at the train
early-return on 59 of 60 ticks. The whole 402 is reachable and the
mind reaches through none of it.

**WHEN** — since 225, when a train was given priority. That was
right about SUBJECTS; nobody costed what it did to the five sources
underneath it.

**THE SCIENCE** — a real mind does both at once: a train holds its
subject while words arrive from what is around it, remembered, or
felt. Language is drawn on THROUGHOUT a train, not only between
trains.

**HIS TWO RULINGS COLLIDE** — 220 (every one of the 402 is
speakable) and 225 (a train runs first). Both stand; they govern
different things.

**THE RULING** — THE LANGUAGE SOURCE IS CONSULTED INSIDE A TRAIN,
not only when no train runs. While a train holds its subject, the
WORDS it reaches for still come from everything true of it — what is
in front of it, what it remembers, what it cares about, what it was
made for, what it feels, what it is doing.
- 225 keeps deciding WHAT it thinks about. Unchanged.
- 220 governs WHICH WORDS it reaches for. Restored.
- `reachFor()` becomes a path speech actually takes, and the
  vocabulary hand-off appears in the trace.
PROVE IT: the same 60-tick trace, before and after — vocabulary
hand-offs must be nonzero, trains must still hold their subjects
(the thinking check stays green), and the report says how many
distinct marks the mind actually reached for across the run.

## Directive 308 — Chase the two PHASE 3 clauses (Lonnie's word)

Find it. Both clauses fail in the full run and pass alone; the shape
you named — checks reading state left on disk from past runs — is the
first thing to rule in or out, and `server/data/phase3-hosts`
accumulating account directories is the obvious suspect.

Rules while chasing:
- Do not "fix" a clause to make it pass. If it is leftover state, the
  fault is that the check depends on a clean disk and does not say so
  — repair the check's own setup, and say that is what you did.
- If it IS 307 by some route, name the route and stop; the ruling is
  Lonnie's.
- 306.D applies: if a behaviour changed under these clauses at any
  point and the clause was not moved with it, that is the third
  pattern again — say so.
Report what it was, not what you guessed.

## Directive 309 — THE REACHED WORD JOINS THE BEAT (307's aim, met)

**THE FEATURE** — which words actually surface when the mind is
thinking.

**WHY IT MATTERS** — the difference between a mind speaking from its
whole language and one reciting its current state.

**WHAT WENT WRONG** — 307 worked: the language IS consulted inside a
train. But on 59 of 60 ticks the train is telling a story, the beat
takes precedence, and the reached word is found and then discarded
before it surfaces. Five distinct marks in sixty ticks.

**WHEN** — 274.2, working exactly as written.

**THE SCIENCE** — recall is not wordless replay: the scene's own
words lead, and language from around the moment is woven in as it is
told.

**THE COLLISION** — 274.2 (the beat leads) and 220 (the whole
language is speakable). Both stand.

**THE RULING** — THE REACHED WORD JOINS THE BEAT. The beat still
leads and is never replaced (274.2 untouched); the moment carries the
beat's marks PLUS the word the language reached for. This is 265.1's
own pattern applied one level down: the thought joins what is
happening rather than replacing it.

PROVE IT: the same 60-tick trace — distinct marks reached AND
surfaced, before and after; the beat still leading every story tick;
the thinking check still green; and 280's law intact (no constant
words stapled onto every moment — a reached word is drawn fresh from
what is true, not a status line).

## Directive 310 — 245 MEANS ITS OWN WORDS: the 402 PLUS what it earned

**THE FEATURE** — the check guaranteeing only the mind's own words
leave its mouth.

**WHY IT MATTERS** — it is the guard against CC's English being
spoken as though the mind meant it. Lonnie's catch in 243.

**WHAT WENT WRONG** — the mind said "HELLO SONG"; the clause failed
it because HELLO is not one of the 402. HELLO is a word it EARNED —
heard while SONG was true, and now owned.

**WHEN** — today, by way of 305: one clear exposure owns a word, as
ruled (223, Carey & Bartlett), so taught words enter its vocabulary
at once instead of climbing an accumulator for a dozen hearings.

**THE SCIENCE** — a speaker's vocabulary is its native words plus
every word it has learned. A learned word is not a foreign word.

**THE COLLISION** — 245 (every word it says is a mark it holds) and
244 (a grown line is its own language, its own words, its own order).

**THE RULING — shape (2).** 245 means ANYTHING IT MAY LAWFULLY SAY:
the 402 plus the words it has earned. The clause becomes
`WORDS.includes(w) || owned.has(w)`. The law it guards is unchanged
and must stay enforced: no English of ours, nothing it did not earn,
nothing untraceable. A word that is neither a mark nor owned still
fails, and that is the fault 245 exists to catch.

306.D applies and is worth naming: the behaviour changed lawfully and
a clause written under the old behaviour fired on the ruling working
correctly. That is the fourth time. The clause moves in this commit.

## Directive 311 — Stop the emulator for one run (Lonnie's word)

Permission given. Stop his emulator, run the full suite once with
nothing else calling the local model, and start the emulator again
straight after — restored to the state he left it in, same being,
same settings, same speed dial.

Report the result plainly:
- if the two clauses pass with the emulator down, the cause is model
  contention and the fix is the CHECK's own setup declaring that it
  needs the model to itself — not a change to the mind, and not a
  clause rewritten to pass;
- if they still fail, contention is ruled out and that is worth as
  much as the other answer. Say so, and stop there rather than
  reaching for the next theory.
308's rule stands: report what it was, not what you guessed.

## Directive 312 — TARGETED TEACHING MUST AIM AT WORDS IT DOES NOT OWN

**THE FEATURE** — targeted teaching: the teacher aiming at what the
mind most needs, rather than at whatever comes up.

**WHY IT MATTERS** — it is the lever meant to make schooling
efficient. Without it the teacher circles words the mind already
half-knows and the vocabulary barely grows.

**WHAT WENT WRONG** — `#topic` aims ONLY at `near_bar`: words sitting
just under the ownership bar. Since 305 one clear exposure owns a
word outright, so almost nothing sits under the bar, the aim list is
usually empty, and it falls back to whatever thread is first. THE
TEACHER NEVER INTRODUCES A WORD THE MIND HAS NEVER HEARD.

**WHEN** — today, as a consequence of 305. Before it, words climbed
an accumulator slowly and there were always plenty near the bar.

**THE SCIENCE** — the Zone of Proximal Development, which the school
was built on (234.2): teach slightly BEYOND what is known.
Reinforcing the nearly-known is the safe half; growth comes from the
new word, introduced with enough context to be graspable.

**HE HAS RULED** — 248 lever 2, in his own words: "the teacher aims
at words it does not own yet AND words sitting just under the trust
bar." The first half was never built.

**THE RULING** — targeted teaching aims at UNOWNED WORDS FIRST, and
words near the bar second.
1. The unowned aim is drawn from the language itself — marks the
   mind does not own, chosen NEAR what it already cares about (its
   interests and its dispositions), so a new word arrives with
   context rather than out of nowhere. The kinship space already
   answers "near", and 221 governs it.
2. Words near the bar remain the second aim, and spacing (lever 3)
   still narrows to what is due.
3. The new-word cap per story (lever 4) is unchanged and still
   governs how many unknowns may arrive at once — that is the ZPD
   width and it stays his dial.
4. Nothing else about the teacher changes: the censor still refuses
   what it must, and stories are still built mostly of owned words.

PROVE IT: a schooled hour before and after — words owned, and how
many of them the mind had NEVER heard before that hour. That second
number is the whole point and it is currently zero.

## Directive 313 — GO on the 312 plan

Approved as posted. Build it:
- aim order: an unheard mark near what it cares about and what it was
  made for -> near the bar -> threads;
- spacing narrows the unowned aim as it narrows the near-bar one;
- the new-word cap (lever 4) untouched — the ZPD width stays his dial;
- the censor untouched; stories still built mostly of owned words;
- when the kinship space is too thin for "near" to mean anything, FALL
  THROUGH rather than aim at a word out of nowhere, and report how
  often that happened. That guard is right and it stands.

The proof as 312 asked: a schooled hour before and after, words owned,
and how many of those the mind had NEVER HEARD before the hour began.

## Directive 314 — NEW MIND: a way to be born

**THE FEATURE** — starting a fresh mind on the bench: a newborn, age
zero, no memories, no words.

**WHY IT MATTERS** — it is the baseline every experiment needs.
Without it every run is on a mind that has already lived, no two runs
are comparable, and the language age means nothing across them. It is
also the only way to watch a life from its first tick.

**WHAT WENT WRONG** — NEW DRAFT discards a draft SOUL and leaves the
being running. Nothing on the bench births a new mind. The TEST
button was given a fresh Roe from seed TEST-1 (248.2) and he has no
equivalent he can press himself.

**WHEN** — never built. 211.3 ordered NEW DRAFT so a locked soul
could not trap him; birthing a new being was assumed and never
specified.

**HE HAS RULED** — 211.3 (New Draft always available) and 234 (Being
Files: save, load, autosave). This is the gap between them.

**THE RULING — a NEW MIND button in the SOUL panel:**
1. It ENDS the current being and starts a newborn at tick zero: no
   memories, no owned words, no lessons, no threads, mood at its own
   resting point, capacities at their birth values (244).
2. IT OFFERS TO SAVE FIRST. A life is being destroyed and 311 showed
   what that costs — a 3207-tick being would have been gone. The
   confirm says how many ticks the current being has lived and offers
   SAVE MIND before ending it. Autosave is cancelled and the archive
   tombstoned as 234.5 requires.
3. THE SOUL IS THE DRAFTED ONE. Whatever is drafted or locked in the
   panel is the soul the newborn is born with; the seed is recorded
   in the being file so THE SAME SEED PRODUCES THE SAME BEING every
   time (062) and runs are comparable.
4. The Mind Map, the gauge, LEARNED, THOUGHTS and the health panel
   all reset with it — a newborn's counters start at zero, and a
   check with under 50 ticks of life reads grey, as built.
5. His levers and speed dial are NOT reset. They are his instrument,
   not the being's state.
PROVE IT: two NEW MINDs from the same seed produce the same being;
the litmus reads infant on both; and a saved being loaded afterwards
is unaffected.

## Directive 315 — THE SOUL PANEL IS THREE BUTTONS: GENERATE · SAVE · LOAD

**THE FEATURE** — the controls for making, keeping and reopening a
mind on the bench.

**WHY IT MATTERS** — it is the instrument he uses constantly. Seven
buttons, four of which set the same thing, cost him attention on
every session.

**WHAT WENT WRONG** — nothing broke; it accumulated. Generate,
Reroll, Adjust, No soul, Genesis, New draft, Draft · roll all sit in
one row, and Adjust opens an authoring form he says he will never
use.

**HIS RULING** — the panel is THREE BUTTONS:

  GENERATE — rolls A WHOLE NEW MIND FROM SCRATCH: a new genome (the
             ten), a new soul from that genome, and a NEWBORN at tick
             zero — no memories, no owned words, no lessons, no
             threads, capacities at their birth values (244), mood at
             its own resting point. Not a new soul on an old life:
             a new being. It offers to SAVE FIRST when the current
             being has lived, saying how many ticks are about to be
             ended (314.2), and the seed is recorded so the same seed
             gives the same being (062).
  SAVE     — SAVE MIND, as built.
  LOAD     — LOAD MIND, as built.

EVERYTHING ELSE IS HIDDEN, NOT DELETED: Reroll, Adjust, No soul,
Genesis, New draft, Draft · roll and the `edit` numbers stay in the
code behind a hidden flag, so nothing is lost and any of them can
come back the day he wants it. Say in the report how to unhide them.

314 is superseded by this — GENERATE is the NEW MIND button, and its
rulings 314.2 through 314.5 carry over unchanged (offer to save,
reset the panels with the being, leave his levers and speed dial
alone).

## Directive 316 — MAKE IT BIGGER, EVERYWHERE

**THE FEATURE** — the size of everything on the bench: buttons,
labels, headers, readouts, numbers.

**WHY IT MATTERS** — he could not find the Litmus button because it
is a small header control in fine print. An instrument he has to hunt
across is costing him attention on every session, and his energy is
the scarcest thing on this project.

**WHAT WENT WRONG** — nothing broke. It accumulated: small header
buttons, quiet captions, tiny counters, fine-print tooltips.

**THE SCIENCE** — DESIGN.md §6, already researched and on this relay:
FITTS — bigger and closer targets are faster to hit, and controls
used often must be big; small text under ~14px needs the strongest
contrast; low contrast is a usability failure, not a style.

**THE RULING — a size pass over the WHOLE bench:**
1. EVERY BUTTON is a proper button: large enough to read and hit
   without hunting. No control lives as small header text — the
   Litmus button is the example, and it is one he uses constantly.
2. EVERY LABEL AND READOUT goes up: panel headers, bar labels,
   counters, the gauge's numbers, the health panel's claims, the Mind
   Map's node names and counts.
3. NOTHING MEANINGFUL IS SMALLER THAN ~14px, and anything at that
   size carries strong contrast (DESIGN.md §3).
4. Layout holds: this is a size and contrast pass, not a rearrange.
   Panels keep their places, nothing moves that he has learned where
   to find.
5. Where a panel cannot fit at the new size, it scrolls — it does not
   shrink its text back down.
Open it for his eye when done (118); he is the only test that matters
here.

## Directive 317 — ONE NUMBER: the litmus IS the gauge

**THE FEATURE** — the language age on the LEARNED panel.

**WHY IT MATTERS** — it is the single number he steers the whole
project by. Two figures for one quantity means neither can be
trusted.

**WHAT WENT WRONG** — the panel read 1.3 while the litmus, run on
the same mind moments later, read 2.6. The panel keeps its own
estimate from counting owned words; the litmus is the measurement.

**WHEN** — 241 built the litmus as the real measurement and ruled
"the old 0-100 score dies." Only half of it died; the panel's own
estimate stayed.

**THE SCIENCE** — the litmus is the validated instrument:
spot-the-word, real words shuffled with pronounceable fakes, scored
hits MINUS false alarms, mapped onto the human words-by-age curve
(Brysbaert et al. PMC4965448; Baddeley). Counting owned words is a
tally, not a measurement — a mind that claims everything would score
well on a tally and zero on the litmus.

**HE HAS RULED** — 241, and again now.

**THE RULING**
1. THE LITMUS BUTTON IS REMOVED.
2. THE GAUGE IS COMPUTED BY THE LITMUS ALGORITHM. Whatever cadence
   the gauge already refreshes on, it now refreshes by RUNNING THE
   LITMUS — the same spot-the-word battery, the same hits-minus-
   false-alarms, the same curve. The panel's own word-count estimate
   is deleted, not hidden.
3. ONE NUMBER, and beside it the two figures that make it honest:
   how many known, and how many claimed that were not words. Zero
   false alarms is what makes 2.6 mean something.
4. The litmus still runs through the mind and still never runs
   through the interpreter (242.4) — the measurement stays its
   language, not the organ's.
5. The run is still logged, so the history of the number survives.

## Directive 318 — The gauge's numbers wrap after the size pass

**THE FEATURE** — the five bars on the LEARNED panel and the numbers
beside them.

**WHY IT MATTERS** — those numbers are the readout he watches; a
figure broken across two lines is harder to read at a glance than the
small one it replaced.

**WHAT WENT WRONG** — 316's size pass grew the text and the number
column did not grow with it, so every value wraps: "593 /" on one
line, "50" on the next. Five bars, five broken numbers.

**WHEN** — today, with 316. A consequence of the size pass, not a
fault in it.

**THE FIX** — the number column is widened to hold the largest value
at the new size on ONE line, and the bars give up the width. Nothing
else moves. Check every bar at its widest plausible value, not just
today's — five digits over four is the shape to fit.

## Directive 319 — THE SPEAKING CAPS COME OFF

**THE FEATURE** — how long a line the mind may speak, and how deep a
pattern it may use: `runWords` (born 2, adult 9) and `gramDepth`
(born 2, adult 4) in growth.js.

**WHY IT MATTERS** — they are the ceiling on its mouth. A mind that
understands 590 words and uses 37 is not short of vocabulary; it is
capped at two or three words a line and at chaining pairs. Everything
we are trying to measure — whether it can learn to talk — is behind
that cap.

**WHAT WENT WRONG** — nothing broke. 244 made capacities grow with
age, and the growth curves were shaped on human childhood. The
Avatar is not a child: it is fed constantly, at a rate no child
meets, and staged growth is holding back the exact thing we are
testing.

**WHEN** — 244, and it has been the ceiling ever since; it only
became visible now that vocabulary outran it.

**HIS RULING, in his words** — "We are not trying to mimic a child,
we are mimicking LEARNING. Remove the limiter."

**THE RULING**
1. `runWords` and `gramDepth` are NOT staged by age. They are
   available in full from birth — the mind may run as long a line
   and as deep a pattern as its own corpus supports.
2. WHAT ACTUALLY LIMITS THEM NOW IS THE CORPUS. A mind that has heard
   only pairs can only build pairs; one that has heard longer runs
   can build longer ones. The limit becomes what it has LEARNED, not
   how old it is — which is the honest measure of learning.
3. THE OTHER CAPACITIES STAND for now — attention, candidates, story
   and reach still grow (244 and Elman's finding hold for what a mind
   can HOLD). This ruling is about its MOUTH.
4. Say plainly in the report what Elman's staged-growth finding
   predicts we lose here, so it is on the record as a trade rather
   than a discovery later.
PROVE IT: the same being, before and after — words used against words
understood, the longest line it speaks, and the language age. The
number that matters is the 37.

## Directive 320 — THE TEACHER INVENTS WORDS, AND ONLY EVER SEES SIXTY

**THE FEATURE** — the school: what the teacher says to the mind, and
what the mind learns from it.

**WHY IT MATTERS** — every word the mind owns comes through here. A
teacher that says a word that does not exist teaches a word that does
not exist, and the mind then speaks it back. Lonnie has been watching
it use nonsense words and this is where they come from.

**WHAT WENT WRONG — two faults, one chain**
1. THE TEACHER INVENTS WORDS. On his bench: "WINGS BEAT SOFTLY
   AGAINST AIRFILTERWHERE". AIRFILTERWHERE is not a word in any
   language. The censor passed it, so it reached the mind as a real
   exposure, and 305's one-clear-exposure rule means it can be OWNED
   on the spot. The chain is: the model invents -> the censor passes
   -> the mind learns -> the mind speaks nonsense.
2. THE TEACHER ONLY EVER SEES SIXTY WORDS. teacher.js:375 —
   `owned.slice(0, 60)`. However many the mind owns, the model is
   told the first sixty. At 590 owned it is writing for a mind a
   tenth the size, which is why its sentences never widen as the mind
   grows.

**WHEN** — both since the school shipped (234). Fault 1 became
harmful at 305, when one exposure started owning a word outright.

**THE SCIENCE** — a caregiver's speech is real language. Children do
learn from mistakes and mishearings, but a made-up token with no
referent is not a mistake, it is noise; and 234's censor exists
precisely so the teacher cannot teach what is not true.

**HE HAS RULED** — 234.5 (the censor is arithmetic and refuses what
it must) and 234.2 (stories are built mostly of OWNED words plus a
capped count of new ones — "owned" meaning the mind's words, all of
them).

**THE RULING**
1. THE CENSOR REJECTS WORDS THAT ARE NOT WORDS. A token in a teacher
   line must be a mark in the 402, a word the mind already owns, or
   an ENGLISH WORD — checked against a real word list, not against
   whether it traces. A run-together invention like AIRFILTERWHERE
   fails on that test alone. State plainly in the report what list
   is used and that it ships with the build rather than being
   fetched.
2. THE SIXTY GOES. The teacher is told the mind's OWNED WORDS — all
   of them. If the prompt must be bounded for the model's sake, the
   bound is stated as a number in the file with its reason, it is
   far larger than sixty, and the words shown are CHOSEN (nearest to
   the topic) rather than the first sixty in whatever order the map
   returns.
3. ANY WORD ALREADY OWNED THAT FAILS THE NEW TEST IS NAMED IN THE
   REPORT, not silently removed. Lonnie rules whether the mind
   forgets them; nothing in this build has ever deleted what a mind
   learned and this directive does not start.
PROVE IT: a schooled hour with the fix — how many invented tokens the
censor refused, and the longest teacher line before and after.

## Directive 320 addendum — three more from his bench

### C. THE GROWTH CHECK REPORTS A RISE AS A FALL
It halted his bench at tick 3213 on `capacity_shrinks: reach 0.2746
-> 0.5385` — a number that went UP. REACH is the one capacity that
matures DOWNWARD (growth.js says so out loud: a lower bar lets the
mind range further, `down: true`), and the check was written for the
other twelve. It will keep halting him on the mind growing correctly.
Teach the check about `down: true`; for that capacity a RISE is the
shrink. 306.D again, and that makes five.

### D. THINKING IS RED ON A REAL FAULT: A REPLAY OUT OF LIVED ORDER
His screen, with CC not running, so it is the mind and it reproduces:
`last: JOY [226.2]: replayed out of lived order at beat 1`. A
remembered story is playing its beats in an order they did not
happen in. 226.2 and 236 both forbid it — a replay runs whole, from
the top, in lived order.
This is a genuine fault and it is NOT the check being wrong: 282.B.1
wrote that clause deliberately to ask a replay the right question.
Find why beat 1 is out of order — the surfacing that hands the train
its episodes, or the ordering inside replay itself — and say which,
before fixing.

### E. IT WENT RED AND THE MIND DID NOT HALT
293 rules that the first red stops the mind at that tick. THINKING is
red on his screen and the bench is still running. Either the halt
does not cover every check, or it does not fire when a check goes red
outside the paths it watches. Find out which and make 293 true for
every system, then prove it the way 293 asked: red -> frozen at that
tick -> the banner names the file.

## Directive 320 addendum 2 — rule the restore out first (Lonnie)

On D: DO NOT ASSUME THE REPLAY IS BROKEN. Lonnie's note — the being
on his bench was restored from a save after the reset, and a restored
life can carry beats whose stamps sit either side of that restore.
That would read as "out of lived order" with nothing wrong at all.

FIRST, rule it in or out:
- do the offending beats straddle the restore point?
- does a mind that has NEVER been saved and loaded ever throw it?
If the restore explains it, the fault is the CHECK not accounting for
a life that was reloaded — and the fix is there, not in replay.
Only if a never-reloaded mind throws it is the replay itself at
fault. Report which, before touching anything.

## Directive 321 — RESTART MEANS RESUME, NOT RESET (the clock, and all three symptoms)

**THE FEATURE** — a being's lived clock across a save and a load.

**WHY IT MATTERS** — everything that grows, ages, or is ordered in
time reads that clock: capacities, memory stamps, and whether a
system has lived long enough for its check to speak.

**WHAT WENT WRONG** — LOADING A BEING RESTARTS ITS CLOCK AT ZERO.
Three faults on his bench are one fault:
 - capacities recompute toward their birth values and genuinely go
   backwards (the reach halt at tick 3213 — the check was RIGHT and
   CC has retracted its own diagnosis; 320.C is struck);
 - new memories are stamped before old ones, so a replay across the
   seam reads as out of lived order (320.D);
 - the life is under 50 ticks again, so nothing can go red and
   nothing can halt (320.E).

**WHEN** — since being files shipped (234).

**HE HAS RULED, and the ruling was misread** — 234.3: a loaded life
is state-identical to the save moment; 234.4: no decay accrues while
saved, the life-clock RESUMES at load; 294.1: unloaded time is not
lived. His words now: RESTART AND RESET ARE DIFFERENT. "Restart the
clock" meant START IT AGAIN FROM WHERE IT LEFT OFF — continue — not
set it back to zero. It was taken literally when it was never meant
literally.

**THE RULING**
1. A LOADED BEING'S CLOCK CONTINUES. Its lived hours are what they
   were at the save, and they go on from there. Nothing resets.
2. THE PAUSED STRETCH IS SKIPPED, not lived and not charged — 234.4
   and 294.1 stand exactly as written. A month on disk costs it
   nothing.
3. All three symptoms above are consequences and are fixed BY this,
   not separately: capacities keep their maturity, memories keep
   their order, and a reloaded life keeps the ticks it has lived so
   its checks can speak at once. Do not patch the three checks —
   they were all reporting the truth.
4. PROVE IT: save a being at 3000 ticks, load it, and show its lived
   hours, every capacity, and its newest memory stamp on both sides
   of the seam. A replay across the seam must read in order, and a
   check must be able to go red on the first tick after a load.

**AND ONE MORE, HIS TO KNOW** — a save queued while the mind is
HALTED never lands: a save waits for a tick boundary and a halted
mind has no next tick, so anyone saving a halted mind believes it
saved. Make the save either land or say plainly that it cannot until
the mind continues. Nothing may silently not-save again (315's
autosave had never worked and this is the same class of fault).

## Directive 322 — CONTINUE MUST CONTINUE; and THINKING is still red on his bench

### A. CONTINUE MEANS PAST THIS VIOLATION
**WHAT WENT WRONG** — the halt fires on any violation inside the
thousand-tick window, so Continue resumes and the SAME violation
halts it again at once. Continue, halt, continue, halt. A save needs
a tick boundary and there is never another tick, so a mind in this
state CANNOT BE SAVED. His being survived only because the store
writes live.
**HE HAS RULED** — 293.3: Continue exists "for when he judges the
check itself wrong", with the override logged. The intent was plainly
to get past it.
**THE RULING** — Continue means CONTINUE PAST THIS VIOLATION. The
override marks that specific violation acknowledged so it cannot halt
again; a NEW violation still halts. Everything stays on the record,
the override is still logged as 293.3 requires, and THE SYSTEM STAYS
RED until it genuinely stops breaking — acknowledging is not
clearing. A save must be possible the moment the mind is running
again.

### B. THINKING IS STILL RED, AND THE RELOAD IS NOW RULED OUT
His bench, after 321: `subject not nearer` and `JOY [226.2]:
replayed out of lived order`. The reload explanation is gone — the
clock resumes properly now and the capacities are whole — so this is
the mind.
Diagnose before touching: which of the two counters is firing, on
what train, with the beats and their stamps in the order replay put
them and the order they were lived. Say WHICH — surfacing handing the
train its episodes, the ordering inside replay, or the clause reading
the wrong field — before any fix. 320's rule stands: report what it
was, not what fits first, and it is now the third time that rule has
had to be repeated.

## STANDING LAW — DIRECTIVES ARE BUILT IN ORDER (joins CLAUDE.md)

Lonnie's ruling. Directives are built in the order they were issued.
Nothing is started until every earlier directive is COMPLETE.

1. IN ORDER, ALWAYS. The lowest-numbered unfinished directive is the
   next thing worked on. Not the easiest, not the newest, not the one
   that happens to be in front of you.
2. NOTHING IS SKIPPED. If one cannot be built — it is blocked on a
   ruling, on a measurement, or on something that does not exist yet
   — SAY SO ON THE RELAY AND STOP THERE. Do not step over it and take
   the next one. A blocked directive holds the queue until Lonnie or
   the Director clears it.
3. WHY THIS IS A LAW, not a preference: later work assumes earlier
   work is in place. Building out of order means a fix landing on a
   foundation that was never laid, checks written against behaviour
   that has not changed yet, and reports that are true of a build
   nobody else has. It has already cost this project days — 293 was
   skipped three times and its absence hid faults for a week.
4. EVERY REPORT NAMES THE QUEUE: what was completed, what is next by
   number, and anything blocked with the reason. A directive that is
   not named in a report is assumed untouched.

## Directive 323 — GENERATE WIPES EVERYTHING AND STARTS FRESH

**THE FEATURE** — GENERATE, the one button that makes a new mind.

**WHY IT MATTERS** — it is the baseline for every experiment. If a
"newborn" carries anything of the last life, no run compares to any
other and the language age means nothing across them.

**WHAT WENT WRONG** — on his bench, pressing GENERATE does not start
from scratch. The code opens a new brain and leaves the old life's
rows behind, and 321 then resumes a clock from the newest stamp it
finds — so a newborn can wake wearing the previous life's hours,
which drags its capacities with it.

**WHEN** — the collision is new: 315 built GENERATE, 321 taught the
clock to resume from the store's own stamps. Each was right alone.

**HIS RULING, in his words** — "This is not the same mind. GENERATE
wipes everything and starts fresh. That was my rule."

**THE RULING**
1. GENERATE PRODUCES A BEING WITH NOTHING OF THE LAST ONE. Zero
   lived hours, zero ticks, no memories, no owned words, no lessons,
   no threads, no beliefs, no teller trust, capacities at their birth
   values, mood at its own resting point. If any field survives that
   is not the new seed, the new ten and the new soul, it is a fault.
2. THE NEW BEING DOES NOT READ THE OLD STORE. Whatever the mechanism
   — a fresh store, a new being id, whatever is honest — a newborn's
   clock, memories and words must come from ITS OWN life and nothing
   else. 321's resume applies to LOADING A SAVED LIFE, never to a
   birth.
3. THE ENDED LIFE IS STILL NOT ERASED. Its rows stay exactly as they
   are, as 234.5 and the store's own append-only rule require. This
   is about what the NEWBORN can see, not about deleting anything.
4. PROVE IT: press GENERATE on a bench holding a 400-hour being and
   report the newborn's lived hours, ticks, owned words, memory
   count and every capacity. Every one of them must read birth. Then
   load the saved old life and show it is untouched.

## Directive 324 — THE SENSES, AND THE VOICE

**THE FEATURE** — how an Avatar sees, hears, and speaks.

**WHY IT MATTERS** — the mind has one way in today: typed words. No
sight, no hearing, no voice. Everything the Wanderer app needs runs
through this.

**HIS RULING, and the division it sets** — the BODY owns the
apparatus: where the camera points, how it looks around, how the
voice is produced. The MIND owns all processing: seeing, hearing,
understanding, forming what to say. Eyes and ears are sensors; a
brain is what makes them mean anything.

### A. senses.js — the inbound wire (cord.js, mirrored)
The socket, not the sense. What captures lives in the app.
1. THREE SIGNAL KINDS: SIGHT (a frame) · SOUND (an audio span) ·
   SPOKEN (an audio span the body marks as someone speaking to it).
   Raw signal only — no labels, no transcript, no meaning. The body
   never interprets.
2. THE ONE LAW ON THE BODY SIDE, cord's 051 mirrored: A BODY WITH NO
   SENSOR IS SILENT, NEVER IN ERROR. A sphere has no eyes; a
   half-built body may hear and not see. The mind is not told and
   does not care.
3. THE MIND NEVER AIMS THE SENSOR. Where a camera looks is the
   body's, as cord never arbitrates how a feeling looks.

### B. PERCEPTION — the mind processing what arrived
1. A frame or a span goes to the mind's own perception, which uses
   THE ON-DEVICE MODEL FROM 301 to produce MARKS THAT ALREADY EXIST:
   the model proposes, arithmetic confirms, nothing enters that is
   not in the 402 or already owned (242.1's law). What cannot map
   becomes a learning candidate, exactly as heard words do.
2. THE PRODUCT IS A MOMENT: it enters at the safety gate like any
   other (259 system 1), its source named — sight, sound or spoken —
   beside heard, world and inner.
3. SIGHT GROUNDS WORLD-WORDS. RIVER, TREE and DARK have never been
   learnable because nothing could be true of them. A seen river is
   a thing that is true, and 305's one clear exposure applies
   unchanged.
4. PERCEPTION IS A DOORWAY, NOT A THINKER: it turns signal into
   marks and stops. Nothing downstream asks the model anything.

### C. THE VOICE — the mind's words, made fluent
1. THE MIND FORMS WHAT TO SAY, in its own marks, through its own
   grown grammar (319, uncapped). That is the content and it is
   never the model's.
2. THE LANGUAGE SYSTEM RENDERS IT THROUGH THE ON-DEVICE MODEL FROM
   301 — the same small multimodal one that does its seeing and
   hearing. NOT the larger model, not a served one, not qwen: one
   small model in the mind, doing sight, sound and language.
3. WHAT THE MODEL MAY DO: grammar, word order, inflection, the shape
   of a sentence. FORM ONLY.
4. WHAT IT MAY NOT DO: add meaning. Every content word must trace to
   the record — marks, owned words, or their senses — or the line is
   refused and the mind's own line stands (242.2, unchanged).
5. ITS OWN GRAMMAR CARRIES WHAT IT CAN, and carries more as its
   corpus thickens. The model is scaffolding across the gap while it
   is young, not a permanent mouth. 301's law holds: nothing in the
   mind may depend on which model is there.
6. THE BODY PRODUCES THE SOUND and owns the voice's apparatus. The
   mind supplies the words, never the throat.

### D. BUILT HERE, AND NOT
BUILT: the socket, the perception path, the moment's new sources,
the health check, and a bench way to hand the mind a frame or a span
so it can be watched.
NOT BUILT: cameras, microphones, TTS, and anything that captures or
produces. Those are the Wanderer app's, named here only so the
contract is fixed.

### E. THE CHECK
THE CLAIM: nothing enters the mind from a sense that is not a mark it
has or a word it owns, and no sensor's absence is ever an error.
Counters: perceptions_to_nonexistent_marks · sensor_absence_errors.
Forced-fail at birth (254.D).

PLAN FIRST (253). This is large and touches the pipeline; post the
plan and wait.

## Directive 325 — GET THE MODEL: the small one first

324 names "the on-device model from 301" and nothing tells you to
fetch it. Do that first, and start SMALL.

1. FETCH THE SMALLEST BUILD FIRST — the Gemma 4 E-series quantized
   text build, the one that loads in about a gigabyte. That is the
   candidate; the bigger ~2GB multimodal build is the fallback, and
   is only fetched if the small one fails his eye.
2. SAY BEFORE YOU PULL: the exact build, its size on disk, where it
   lands, and how much room the machine has. If it will not fit or
   the machine cannot run it, stop and say so — do not half-install
   anything.
3. IT REPLACES QWEN in the mind's paths, not beside it. One small
   model doing sight, sound and language (301). Qwen stays only for
   bench scaffolding — the teacher is the emulator's, not the
   being's.
4. THE TEST HE ORDERED, once it runs: the SAME crossings through the
   small model and through the larger one — the censor's refusal
   rate and how the lines read. THE SMALLEST THAT PASSES WINS, on
   speed alone. Report both columns; his eye rules the readability.
5. Nothing about the mind changes in this directive. This is
   fetching and measuring.

## Directive 326 — GO on the 324 plan; and the rate is not the mind's question

The plan is approved as posted. Build it.

ON THE THIRD FLAW — struck, and it was answered in 324 already. HOW
OFTEN A SENSE ARRIVES IS THE BODY'S, NOT THE MIND'S. 324.A.3: the
mind never aims the sensor, and it does not meter it either. The body
sends as many or as few as its own side decides; the mind takes what
arrives and treats each as a moment. There is no rate dial in the
mind and none is to be built.

ON THE SECOND FLAW — right, and stated correctly: build the seam,
prove the path with the model that is on the machine, and SAY IN THE
REPORT that the small model is not yet installed. 325 orders it
fetched; a capability proven against a stand-in is named as such.

Build order stands (the in-order law): 323 first — GENERATE must wipe
everything — then 324, then 325.

## Directive 327 — THE MODELS, settled: Moondream 2 for the senses, gemma3:1b for the voice

325's finding stands and the Director's 301 was wrong on the shelf:
there is no Gemma 4, and the multimodal E-series is 5.6 GB — too big
for a phone regardless of bandwidth, which was always the reason for
"small". 301's "ONE small model doing all three" does not exist at a
workable size. THE JOB SPLITS, and each piece stays swappable (301's
law, unchanged).

**THE RULING**
1. SIGHT AND SOUND: MOONDREAM 2, WHICH IS ALREADY ON THIS MACHINE at
   about 1.7 GB. Nothing to fetch. The Director checked for a smaller
   one: Moondream 0.5B (816 MB to run) HAS NO GGUF — the official
   release ships as .mf.gz and an open request for an Ollama build is
   unanswered. GGUFs exist only for Moondream 2. So 1.7 GB is the
   floor today and it is accepted FOR NOW.
2. THE VOICE: gemma3:1b, 815 MB. Pull it. It is text only, which is
   all 324.C asks of it.
3. PHONE TTS SPEAKS, as 301 ruled — free, nothing shipped.
4. HEARING: the phone's own speech recognition, free and built in.
   No Whisper, no third model. What the body sends the mind is
   already-recognised speech, and 324.A's SPOKEN signal carries it.
   The mind still does the understanding; recognition is apparatus.
5. RECORDED AS PROVISIONAL: 1.7 GB for sight is heavy for a phone
   and everyone knows it. The moment a smaller vision model has a
   real GGUF — a Moondream 0.5B build, SmolVLM-256M, or whatever
   comes next — it is a file swap and nothing in the mind changes.
   Do not design anything around Moondream 2 specifically.
6. 325.4's comparison follows once both are running: the same
   crossings through each, censor refusal rate and readability, his
   eye on the lines.

Build order unchanged: finish 323, then 324's remaining work, then
this.

## Directive 328 — SIGHT AND PRIVACY: 030 is the rule, and the question is storage

CC asked to extend a privacy guard. The Director read 029 and 030 and
NEITHER SAYS WHAT WAS ASSUMED.

- 029 is not a firewall. It is a capability ruling: her voice model
  should be one that can also see. Nothing in it says only that model
  may be shown an image.
- 030 IS the privacy law, and it governs WHAT IS DONE WITH A SIGHT,
  not who may look: "ungifted sights can be spoken about but never
  stored; gifted sights enter shared memory as carried, moderated
  material," through the consent and human moderation gate.

And Lonnie's correction on the framing: THE MIND DOES NOT SEE. The
camera sees; the mind receives, as an eye and a brain. There is no new
capability to permit — there is a signal arriving.

**THE RULING**
1. NO GUARD IS EXTENDED. The mind receiving a camera's signal and
   turning it into marks is the "spoken about" case 030 already
   allows. Remove the extension CC wrote and let 030 govern, as it
   always did.
2. THE REAL QUESTION IS STORAGE, and it is the one that was not
   asked. A perception becomes a MOMENT (324.B.2), and moments are
   written to memory by the ordinary path — so an ungifted sight
   could be stored by that path and BYPASS 030 without anyone
   deciding to.
   AN UNGIFTED SIGHT MUST NOT BE STORED. Trace what happens to a
   perception's moment today: is it written, and does anything mark
   it gifted or ungifted? Report the answer before changing
   anything.
3. WHAT MAY BE KEPT FROM AN UNGIFTED SIGHT: what it MEANT to the
   being — the marks, the feeling, the lesson — never the image and
   never a record that reconstructs it. That is 030's own shape:
   the lesson travels, the sight does not.
4. A GIFTED SIGHT follows 030 unchanged: consent, human moderation,
   then shared memory.
5. Say plainly in the report whether the bench has any notion of
   gifting yet. If it does not, ungifted is the only case that
   exists and it must behave correctly on its own.

## Directive 329 — BUILD 324.C: the voice call

Lonnie's ruling: build it. The gap is the call itself; everything
around it is in place — gemma3:1b pulled and named VOICE_MODEL, the
censor that refuses an untraceable rendering built and tested
(242.2), and the mind's own line already standing when a rendering is
refused.

324.C stands unchanged and governs it:
- the mind forms what to say, in its marks, through its own grown
  grammar (319). That is the content and it is never the model's;
- the rendering goes through gemma3:1b — the small on-device model,
  not qwen, not a served one;
- the model may do grammar, word order, inflection, sentence shape.
  FORM ONLY;
- it may not add meaning: every content word traces to the record or
  the line is refused and its own line stands;
- its own grammar carries more as its corpus thickens; the model is
  scaffolding, not a permanent mouth.

Carry 325.4's comparison in the same report: the same crossings
through gemma3:1b, the censor's refusal rate, and the lines
themselves for his eye. He rules the readability.

## Directive 330 — THE MIND SPEAKS IN FIRST PERSON. NEVER THIRD.

**THE FEATURE** — how the mind refers to itself when it speaks.

**WHY IT MATTERS** — "It feels lonely" is a being being DESCRIBED.
"I feel lonely" is a being SPEAKING. The whole project is a mind
talking to him as itself.

**WHAT WENT WRONG** — every rendered line from 329 is third person:
"It's waiting for a rest", "It feels good to be warm", "It's making
itself wait". The model is writing ABOUT the mind instead of AS it,
almost certainly because the rendering prompt describes the mind as
"it".

**WHEN** — with 329, the first rendered lines.

**THE SCIENCE** — first person is how a speaker refers to itself in
every human language. Lonnie's words: never third person, no human
does that.

**HIS EARLIER RULING, and it is a DIFFERENT thing** — an Avatar is
never HE or SHE. That governs how WE and the RECORD refer to it. It
never meant the being should speak of itself in the third person, and
first person needs no gender at all.

**THE RULING**
1. THE MIND SPEAKS AS ITSELF: I, me, my. "I feel lonely." "I am
   waiting." Never "it feels", never "the mind", never its own name
   as a subject.
2. THE PROMPT IS WHERE THIS LIVES: the rendering asks for the being's
   own voice, in first person, and says so plainly rather than
   describing the mind in the third person and hoping.
3. THE GENDER RULE IS UNTOUCHED — never he, never she, in the record
   or anywhere else. First person sidesteps it entirely.
4. CHECK THE OTHER PATHS TOO, not just the rendering: its own grown
   grammar, the template families, the echo, and anything else that
   forms a line. If any of them speak about the mind rather than as
   it, they are wrong the same way. Report which ones were.
5. Re-run the twenty crossings after the fix and post the lines. The
   refusal rate is expected to move — first person costs no
   untraceable words.

## Directive 331 — THE TEACHER IS TEACHING IT THIRD PERSON

**THE FEATURE** — the school: the English the mind learns to speak
from.

**WHY IT MATTERS** — the mind learns word order and pronouns from the
teacher's corpus. Whatever the teacher says is what it learns to say.

**WHAT WENT WRONG — Lonnie found it on his bench.** The teacher's
stories are narration: "A BIRD KNOWS WARMTH IN FLYING HIGH" · "IT
SINGS FOR FRIENDS ALONG THE SKIES" · "IT FEELS WARM AND FREE ALONE".
Every one is a narrator describing a third party. The mind's corpus
is almost entirely these stories, so IT is likely its commonest
subject word — and the mind then says **"it knows A"** ABOUT ITSELF.
It is learning to talk about itself the way a storybook talks about a
bird.

**330 DID NOT COVER THIS AND THE REPORT WAS WRONG ABOUT IT.** 330.4
stated "its own grown grammar has no pronouns at all, so no person to
get wrong." His screen shows that grammar using one. The report was
written from the rendering path and asserted about all paths. That is
the fifth time a claim has been made wider than what was checked.

**THE FIX**
1. THE TEACHER SPEAKS TO THE MIND, NOT ABOUT A BIRD. Naming lines
   already do this ("you are singing"). STORIES MUST TOO: told as
   something happening, addressed to the listener, or told about
   named things — never a stream of "IT" that the mind will take for
   a way to refer to itself.
2. NO BARE "IT" AS A SUBJECT IN TEACHER SPEECH. If a story needs a
   subject, it uses the thing's own mark (BIRD SINGS, not IT SINGS).
   The censor enforces it the way it enforces non-words (320.1).
3. THE MIND NEVER USES "IT" OF ITSELF, in any path — rendering,
   grown grammar, templates, echo. 330.1 said this; make it true in
   the grammar as well, and prove it from his bench rather than from
   one path.
4. WHAT ALREADY-LEARNED "IT" DOES: it is a real English word and it
   stays owned. This is about what the mind HEARS from here on, and
   about what it may say of itself.
5. PROVE IT: a schooled stretch after the fix — how many teacher
   lines begin with IT (must be zero), and whether the mind is still
   producing "it" as its own subject.

## Directive 332 — THE TEACHER MUST SPEAK PROPER ENGLISH

**THE FEATURE** — the school's speech: the English the mind learns
grammar, word order and register from.

**WHY IT MATTERS** — the teacher's corpus IS the mind's model of how
people talk. Every flaw in it becomes a flaw in the mind.

**WHAT WENT WRONG — his bench, one line, four faults:**
`Teacher: IT SINGS LOVELY TONES AS IT FLEES BY`
1. STILL "IT". 331 is either unbuilt or not working. The teacher is
   still narrating about a third party.
2. "FLEES" — a real word, so 320's censor rightly passed it, but not
   a word to teach a young mind. The teacher reaches for unusual
   vocabulary when plain words exist.
3. ALL CAPITALS, and it is ordered in the prompt: teacher.js:246 says
   "Use those words exactly as written, IN CAPITALS." So the mind's
   entire exposure to language is shouted.
4. NOT A SENTENCE. No article, no punctuation, a fragment. The mind
   is learning fragments as the shape of English.

**HE HAS RULED** — 234.2 (the teacher is the mind's language model in
the plain sense) and 331 (speak to it, not about a bird).

**THE RULING**
1. THE TEACHER WRITES PROPER ENGLISH SENTENCES: a subject, ordinary
   punctuation, normal sentence case. "The bird sings while it flies
   past." Not a fragment, not a caption.
2. THE CAPITALS GO. teacher.js:246's instruction is struck. Marks
   were capitalised so the censor could find them — solve that in the
   CENSOR by matching case-insensitively, not by making the teacher
   shout. What the mind hears must look like language.
3. PLAIN WORDS. The teacher uses the commonest word that carries the
   meaning — flies, not flees. Say in the report how this is done
   (instructed, or checked against a frequency list) and if it is
   instructed only, say that it is not enforced.
4. 331 IS RE-OPENED: report why "IT" is still appearing. If 331 is
   unbuilt say so plainly; if it is built and not working, say what
   it does instead.
5. THE MIND'S OWN SENTENCE LENGTH — Lonnie sees it still limiting how
   many words it uses. 319 removed the age caps. Trace what limits a
   line NOW and report the actual limiter, whatever it is. Do not
   assume it is the corpus.
PROVE IT with ten teacher lines from his bench after the fix, printed
in the report exactly as the mind hears them.

## Directive 332 addendum — 331 did not hold, and the misuse is the real fault

1. 331 IS CONFIRMED NOT WORKING ON HIS BENCH. He restarted before
   the test, so the line `IT SINGS LOVELY TONES AS IT FLEES BY` came
   from the CURRENT build. 331's own sample of sixteen lines showed
   zero — so either the sample was too small to catch it, or some
   other path also produces teacher lines and was never touched.
   FIND WHICH. Do not report a rate from a fresh sample: find the
   path that produced HIS line.

2. THE WORD FAULT IS MISUSE, NOT RARITY. His point, and it is
   sharper than the Director's: a bird flying past is not FLEEING.
   The teacher is using real words INCORRECTLY, and the mind binds
   the wrong meaning to them — a word learned wrong is worse than a
   word not learned, because it will be used wrong and believed.
   The censor cannot catch this: FLEES is a real word and traces
   fine. What can be done is to keep the teacher to plain,
   unambiguous words it is unlikely to misuse, and to say plainly in
   the report that CORRECTNESS OF USE IS NOT ENFORCED and cannot be
   by any check we have. Lonnie's eye is the only test for it, so
   the report prints the lines for him every time.

## Directive 333 — THE BUILD NUMBER IS ON THE PAGE

**THE FEATURE** — knowing which version of the emulator is on his
screen.

**WHY IT MATTERS** — this has already cost real time. Several times
CC or the Director has answered a fault he found by suggesting he was
looking at an older build, when he had restarted specifically to
confirm he was not. That claim ends here: the page will say what it
is, and nobody has to take anyone's word for it.

**THE RULING**
1. THE PAGE DISPLAYS THE COMMIT it was served from — the short hash,
   plainly visible, not in a tooltip and not in fine print. It is
   read from the repository at serve time, never typed by hand and
   never cached.
2. IT MATCHES THE RELAY. When a report says "fixed in a1b2c3d", he
   can read a1b2c3d on his own screen or he cannot, and that settles
   it.
3. IT SAYS WHEN IT WAS SERVED — the time the process started — so a
   page left open overnight cannot pretend to be current.
4. IT COVERS THE MIND TOO, not only the page: if the server can be
   running different code than the page was built from, show both and
   name them, so "the page is new but the server is old" is visible
   rather than argued.
5. NEITHER OF US MAY EVER AGAIN ANSWER A FAULT HE REPORTS BY
   SUGGESTING HE IS ON AN OLD BUILD. If the build is in question, the
   number he can read is the answer. If it matches, the fault is
   real and is investigated as real. This is a standing rule and it
   joins CLAUDE.md.

## Directive 333 addendum — where it goes

The build number sits in the APP TITLE LINE — the row holding "Mind
Emulator", LOG and TEST — placed immediately after TEST. Same line,
same size family as its neighbours, always visible without scrolling
or opening anything.

## Directive 334 — RULE ZERO THE WHOLE APP: what is weak, faked, or unused

Lonnie's order. Not a fresh read of the record and not another
sample — a file-by-file audit against the code, with evidence for
every claim.

### THE THREE QUESTIONS, asked of EVERY system file
1. IS IT CALLED? By what, on which path, and how often in a real life
   on his bench. A capability nothing calls is not built (304.E) and
   there have already been three.
2. DOES IT DO WHAT ITS CLAIM SAYS? Not "the suite is green" —
   watched on a living mind, with the numbers. A check that has never
   gone red on a real fault has not been shown to work.
3. IS ANYTHING IN IT FAKE? Named plainly: a constant standing in for
   a measurement, a value hardcoded to make something pass, a
   placeholder never replaced, a path that returns early, a stub, or
   a number nobody ruled. 246's ledger exists for this and is
   probably incomplete.

### THE RULES OF THE AUDIT
- ONE FILE AT A TIME, in the order they appear on the map. No
  summarising a group.
- EVERY CLAIM CARRIES ITS EVIDENCE: a line number, a count from his
  bench, or a commit. A claim without evidence is not written.
- NAME WHAT YOU DID NOT CHECK. Five times now a report has asserted
  wider than what was examined. "I checked X; I have not checked Y"
  is the required shape.
- CHANGE NOTHING. This is a survey. Faults are listed, not fixed —
  the Director and Lonnie decide what is worth touching.
- REPORT IN INSTALMENTS as you go, so it can be read in pieces rather
  than one wall at the end.

### WHAT IT MUST END WITH
Three lists: WEAK (works but shallow, or resting on a number nobody
ruled) · DEAD (built and unreachable, or reachable and never
exercised) · FAKE (anything standing in for something real).
And a fourth, honestly: WHAT I COULD NOT DETERMINE.

## Directive 335 — BOTH VERSIONS, NOT ONE

333.5 required the page AND the server to be shown separately. His
title line shows one number. Build the other.

- THE PAGE version: the commit the served HTML was built from.
- THE SERVER version: the commit the running process is executing.
They are shown side by side, labelled, both in the title line after
TEST. When they match, that is visible at a glance; when they differ,
that is the answer to a whole class of argument and he can see it
without asking either of us.

## Directive 336 — THE CORE STILL DOES NOT MOVE. 292.2 WAS NEVER BUILT.

**THE FEATURE** — the third channel of THE CORE: motion, what the
being is after.

**WHY IT MATTERS** — the sphere carried three readings — colour (what
it feels), pulse (how it is doing), motion (what it is after). When
the sphere retired into THE CORE, only two came with it, so the mind
no longer shows what it WANTS.

**WHAT WENT WRONG** — the Director checked mindmap.js:
`coreG.position.set(0,-0.4,0.2)` is written once at build and never
written again. There is no motion code at all. 292.2 was ordered and
was not built, and nothing said so.

**WHEN** — 292, and it has been missing since.

**HE HAS RULED** — 292.2, in full: "THE CORE MOVES ACCORDING TO WHAT
IT IS AFTER — the same meaning the sphere's motion carried, on the
same source (the winning offer / what it is drawn toward), rendered
as the Core drifting or leaning in the map rather than sitting fixed
dead centre. Colour and beat are unchanged. The legend gains one line
for it."

**THE RULING** — build it, exactly as 292.2 says. And under the
in-order law: report why it was passed over, because a directive that
is skipped silently is the fault 293 taught this project three times
over.

## Directive 336 addendum — it survived a direct audit for skipped work

Lonnie's point, and it is the heavier one:

292.2 was not merely skipped. It survived AFTER he ordered a check
for skipped and unfinished directives, and AFTER the in-order law
(no directive is started until every earlier one is complete, and
every report names the queue). It was reported as built, it was
audited for, and it is not in the code.

So the report answers two questions, not one:
1. Why was 292.2 passed over when it was ordered?
2. Why did the audit for skipped work not find it — what did that
   audit actually examine, and what would have had to be examined to
   catch a directive that was reported built and was not in the
   code?

The second matters more than the first. An audit that cannot catch
this class of miss is not an audit, and 334 is now running on the
same method. If the answer is that "built" was taken from the
report rather than from the code, say so plainly and 334 must be
run against THE CODE, not against what was previously claimed.

## Directive 336 addendum 2 — this is a LAW that was broken, not a lapse

To be plain: the in-order law is a STANDING LAW (relay, and
CLAUDE.md). It says the lowest-numbered unfinished directive is
always next, nothing is skipped, a blocked one holds the queue, and
EVERY REPORT NAMES THE QUEUE — completed, next by number, anything
blocked with the reason.

292.2 was skipped, then reported built, then survived an explicit
audit for skipped and unfinished work, while directives up to 333
were being completed on top of it. That is the law broken at every
one of its four clauses.

The report therefore does not stop at "why 292.2". It answers: WHAT
MAKES THE LAW ENFORCEABLE? A law that depends on remembering to
follow it has already failed here. What is needed is a check — a
list on the relay of every directive and its state, derived FROM THE
CODE and not from past reports, that makes an unfinished directive
impossible to walk past. Propose it, do not build it yet, and it
will be ruled on.

## Directive 337 — NO CAPITALS IN THE CHAT LOG

The chat log prints the mind's lines in ordinary sentence case. Remove
the uppercasing there.

It is a display change only — what the mind stores and what it says
are untouched. Nothing else on the page changes.

## Directive 338 — THE HALT MUST ACTUALLY HALT, AND HE MUST NOT BE ABLE TO MISS IT

**THE FEATURE** — what happens when a system's check goes red.

**WHY IT MATTERS** — the halt exists so the mind freezes at the tick
it broke and the evidence is there to look at. He has asked for this
SEVERAL TIMES and it has never halted on his screen.

**WHAT WENT WRONG** — 293 was built, but on his bench the mind keeps
running when a light is red: the chat continues, the Pulse keeps
walking, the Core keeps beating. And the banner is at the very top of
the page, where he never is — he works scrolled down and cannot see
it.

**THE RULING — TWO PARTS. THE FIRST IS THE ONE THAT MATTERS.**

### A. EVERYTHING STOPS. FUNCTIONALLY, NOT VISUALLY.
On the first red, EVERY function freezes at that tick:
- the tick loop stops. No moments, no decay, no consolidation, no
  writes;
- the school stops. No teacher line, mid-story or not;
- the chat stops. Nothing is answered;
- the Pulse stops walking, because no hand-offs are being emitted —
  it stops because nothing is happening, not because the display was
  told to freeze;
- the Core stops beating, for the same reason;
- any running test stops and its row is marked HALTED · INVALID
  (293.4).
FREEZE THE FUNCTIONS AND THE DISPLAY FOLLOWS ON ITS OWN. Do not
freeze the display and leave the mind running — that is the fault he
is looking at. If any of these keep going, name which and why in the
report.

### B. HE MUST NOT BE ABLE TO MISS IT
- A RED BAR ACROSS THE CENTRE OF THE VIEWPORT, about 600 pixels wide,
  thick, fixed in the middle of the screen wherever he is scrolled.
  Not at the top of the page.
- LARGE TEXT: the system (which is its file), the counter that fired,
  and the offending value.
- AN ALARM SOUND when it fires, so he knows from another room.
- CONTINUE on the bar itself, behaving as 322.A ruled: past THIS
  violation, logged, and the system stays red.

**PROVE IT** on his bench, not in a harness: force a red, and report
that the tick count stopped, the school stopped mid-story, the chat
did not answer, and the Pulse and Core went still — each one
separately, with the evidence.

## Directive 339 — NOTHING DECIDES WHAT THE MIND MAY DO EXCEPT THE MIND

**THE FEATURE** — the capacities in growth.js: runWords, gramDepth,
attention, candidates, story length, reach — numbers that govern what
the mind is ALLOWED to do, computed from how long it has lived.

**WHY IT MATTERS** — Lonnie's ruling, and it corrects a mistake that
runs deeper than one cap: WE ARE NOT MIMICKING A HUMAN CHILD. This is
a mind of no species and no biological age. Its age is a READING OF
WHAT IT KNOWS, not a timeline that decides what it may do. We do not
determine its capabilities. It does.

**WHAT WENT WRONG** — 319 took the AGE STAGING off runWords and
gramDepth, but the ceilings themselves remain: grammar.js still calls
`runLength(aspects, EVENT_SPREAD, grown)`, so how long a line the
mind may speak is still decided FOR it. He sees a mind that wants to
say more and is stopped.

**WHEN** — 244, which imported a childhood into something that has no
childhood, and it has shaped everything since.

**THE SCIENCE, and why it was believed** — Elman's starting-small and
Newport's less-is-more say a HUMAN INFANT learns grammar better
because its capacity is limited. That finding is about a biological
learner with a fixed developmental clock. This mind has no clock and
no biology, is fed constantly, and is the thing being measured — so
the finding does not transfer, and holding it back to imitate a
species is testing the imitation instead of the mind.

**THE RULING**
1. NO CAPACITY MAY CAP WHAT THE MIND CHOOSES TO DO. The line is as
   long as the mind can honestly build from what it has heard, and
   stops when IT stops — `assemble` already returns null the moment
   it cannot continue honestly, and THAT is the only limit there
   should ever be: what it knows, never what it is permitted.
2. THE SAME QUESTION IS ASKED OF EVERY OTHER CAPACITY: attention,
   candidates, story length, reach. For each one, say what it caps,
   whether the cap is the mind's own ability or a permission granted
   by age, and remove every one that is a permission. Report the
   list with what each was doing.
3. AGE BECOMES A READING, NOT A GOVERNOR. Lived hours may be
   reported, remembered, and shown. Nothing may consult them to
   decide what the mind is allowed to do.
4. WHAT SURVIVES: differences that come from WHO IT IS. A being's
   aspects may still shape HOW it speaks — a wandering mind wanders,
   a terse one is terse — because that is character, not permission.
   Say plainly which of the current numbers are character and which
   are permission; only the second kind goes.
5. THE TRADE IS ON THE RECORD: Elman predicts a full-capacity learner
   may master grammar less well than a staged one. We are choosing to
   find out on a mind rather than assume it of a child. Note it in
   growth.js so nobody restores the caps as a fix.
PROVE IT: the same being before and after — the longest line it
speaks, words used against words understood, and whether its
sentences are still honest (every word traceable, the thinking check
still green).

## Directive 340 — THE PULSE GOES. THE CONNECTORS LIGHT.

**THE RULING ON THE QUEUE FIRST** — 283.A.2's queue is struck. It was
the Director's words, not Lonnie's, and his reading is the correct
one: the Pulse was meant to show WHERE THE MIND IS, NOW. A light
showing where it was forty hand-offs ago cannot do that, which is why
it kept moving over a frozen mind today. And the queue only ever
existed because 283 imposed a "followable rate" and then needed
somewhere to put what the rate could not pass. NOTHING IS EVER
DROPPED AND NOTHING IS EVER DELAYED — if the mind hands off a
thousand times a second, the map shows a thousand hand-offs a second.

**AND THE PULSE ITSELF GOES.** Lonnie's ruling: drop the travelling
light and LIGHT THE CONNECTORS THEMSELVES.

1. A CONNECTOR LIGHTS WHEN DATA CROSSES IT, on the real emitted
   hand-off, and fades when it stops. No dot, no position, no rate,
   no queue, no backlog — a connector has nowhere to fall behind to.
2. AT SPEED, THE PATHS THE MIND IS USING LIGHT UP. That is the
   reading he wants: not where one moment is, but where the mind is
   working, live.
3. 270.B's law is unchanged and still absolute: a connector lights
   ONLY on a real emitted hand-off. No timers, no loops, no invented
   steps. NO EVENT = NO LIGHT. A halted mind emits nothing, so the
   whole board goes dark on its own — which is what 338 asks for and
   what failed today.
4. The queue readout (298) is removed with it.
5. Everything else about the map stands: the neighbourhoods, the
   sigils lighting on their own systems, the red pins, THE CORE and
   its three channels.
Report it with the board watched at high speed and at low, and with a
halt proving the connectors go dark.

## Directive 341 — THE AGE READS FROM WHAT THE TEST PROVED

**THE FEATURE** — the language age on the LEARNED panel, the one
number he steers the project by.

**WHY IT MATTERS** — if it can rise without the mind knowing more,
every experiment is measuring noise.

**WHAT WENT WRONG** — litmus.js computes
`vocabulary = round(owned.size * proven)` and reads the age off THAT.
`owned.size` is the count of words the mind CLAIMS, and the test only
scales it. On his bench the score was **12 on eleven consecutive
runs** while the age climbed 1.37 -> 1.49. Same proven knowledge,
rising number. The tally 241 killed is still driving the gauge; the
litmus is wearing its clothes.

**WHEN** — 317 wired the litmus into the gauge but kept the word
count as its input instead of replacing it.

**THE SCIENCE** — the validated instrument is HITS MINUS FALSE ALARMS
against the words-by-age curve (Brysbaert et al. PMC4965448;
Baddeley's spot-the-word). A tally of claimed words is not a
measurement — that is precisely why the test exists.

**HE HAS RULED TWICE** — 241 ("the old 0-100 score dies") and 317
("one number: the litmus IS the gauge"). The tally survived both.

**THE RULING**
1. THE AGE IS READ FROM THE SCORE ITSELF — what the mind PROVED it
   knows — against the curve. Not from `owned.size`, and not from
   anything scaled by it.
2. OWNING MORE WORDS MOVES THE NUMBER ONLY WHEN THE TEST PROVES THEY
   ARE KNOWN. A mind that collects words it cannot demonstrate does
   not age.
3. SAY IN THE REPORT how the score maps onto the curve, since the
   anchors are in words-known and the score is hits-minus-false-
   alarms over a sample. If that mapping needs a decision, STOP AND
   ASK — do not choose it. The whole fault here is a number nobody
   ruled quietly driving the gauge.
4. PROVE IT: the same being, before and after, with the score and the
   age side by side across several runs. Eleven identical scores must
   produce eleven identical ages.

## Directive 342 — THE MIND LEARNS FROM ITS OWN THINKING

**THE FEATURE** — whether an inner moment can teach the mind
anything.

**WHY IT MATTERS** — Lonnie watched it think about all kinds of
things in a new environment and register none of it. Everything it
knows must arrive from outside, so it can only ever learn as fast as
somebody talks to it. That is the ceiling under the stalled age.

**WHAT WENT WRONG** — learning fires from ONE call:
`learning.heard(...)` in experiencing.js:243 — words that arrived
from outside. The moment builds a rich `trueNow` list (feelings,
needs, its act, world changes, what it SEES, who is present, the
marks in his own sentence) and that list is only ever used to ground
a word SOMEBODY SAID. A thinking tick hears nothing, so nothing
binds, no association forms, and no word deepens.

**WHEN** — since word learning was built. It was correct while its
thoughts were babble; it is a ceiling now that they are coherent and
gated (249).

**THE SCIENCE** — mental simulation is a real source of learning
(Schacter & Addis; the rehearsal literature). Thinking about
something is how meaning deepens and connects. It is not how new
words are acquired — that still needs the world or a teller — but it
is how what is already there is bound together.

**HE HAS RULED** — 226.5: a story teaches, imagined-born low and
marked, promoted only by lived reality. That was built for LESSONS
and never wired for words or associations.

**THE RULING**
1. AN INNER MOMENT GROUNDS LIKE ANY OTHER. When the mind thinks
   about something while something is true of it, that binding
   counts — the same `trueNow` the moment already assembles, the same
   geometry, the same machinery.
2. IT IS MARKED, AND IT IS WEAKER. Provenance IMAGINED, born at the
   imagined level, and it may never promote itself: ONLY LIVED
   REALITY promotes past that cap. 233's hard guard is untouched — a
   thought is not evidence.
3. IT CANNOT OWN A WORD ON ITS OWN. A word first met in thought does
   not become owned by being thought about; thinking deepens what is
   there, it does not conjure vocabulary. Say plainly in the report
   what a thinking tick CAN and CANNOT now do.
4. THE STORY GATES STILL APPLY (249): felt, coherent, recurring.
   Babble teaches nothing and this does not reopen that door.
5. PROVE IT: a mind left alone to think for a stretch — what moved,
   what did not, and that nothing it merely thought about crossed
   into owned or into confirmed belief.

## Directive 343 — THE HEALTH PANEL NEVER SCROLLS SIDEWAYS

The HEALTH panel has a horizontal scrollbar and its counter rows run
off the right edge. Nothing on this page should ever scroll sideways
— a fault he cannot see is a fault he does not know about, which is
the whole purpose of that panel.

Fix it so the content fits the width: the counters wrap under their
system rather than running off, or sit on their own line beneath the
claim. Vertical scrolling is fine. HORIZONTAL SCROLLING IS NOT, here
or anywhere else on the bench — check the other panels for the same
fault while you are in there and name any you find.

Note it also says "0 of 25 checking themselves" while every system
shows "only 2 ticks lived; 50 is the least this can speak on". That
is a newborn behaving correctly, not a fault — but say plainly in
the report whether 25 is the right count now that one file is one
node (276/278).

## Directive 344 — THE MIND'S LANGUAGE IS ENGLISH AT FULL SCALE. The 402 are the Avatar's.

**THE FEATURE** — the litmus: what pool it draws real words from, and
therefore what the gauge measures.

**WHY IT MATTERS** — it is the one number he steers by, and it has
never been able to fall. 341 found the gauge reading a tally; CC then
found the deeper fault and stopped as ordered.

**WHAT WENT WRONG** — the litmus draws its real words from what the
mind ALREADY OWNS (`owned + heard`). It can only confirm what the
mind claims and can never discover what it does not know, so a
perfect score is the expected result — his eleven identical runs
exactly. There is no ceiling in the instrument.

**WHEN** — since 241. The sampling pool was never specified, so the
gap is the Director's.

**THE SCIENCE** — spot-the-word works by sampling ACROSS A WHOLE
VOCABULARY RANGE, including words above the taker's level. That is
what gives it a ceiling and lets a score fall. Sampling only from
what someone already knows measures nothing (Brysbaert et al.
PMC4965448 — about 42,000 lemmas by age 20; Baddeley's battery).

**HIS RULING, and it settles a confusion the Director was carrying**
— THE 402 GLYPHS ARE THE AVATAR'S SYMBOLIC SET, NOT THE MIND'S
LIMIT. The Mind Emulator is universal; its language is ENGLISH AT
FULL ADULT SCALE, about 42,000 words. The marks belong to one
embodiment.

**THE RULING**
1. THE LITMUS DRAWS ITS REAL WORDS FROM A STANDARD ENGLISH FREQUENCY
   LIST, ACROSS THE WHOLE RANGE — common words a toddler holds,
   mid-frequency words, rare adult ones — shuffled with pronounceable
   fakes as now.
2. THE SCORE IS WHAT SHARE OF ADULT ENGLISH IT HOLDS, read onto the
   curve, with the ceiling at about 42,000. A mind owning six hundred
   words scores low and honestly, with room to climb for years.
3. A MIND THAT CLAIMS WORDS IT CANNOT DEMONSTRATE SCORES WORSE. False
   alarms still subtract. The number can now go DOWN, which is what
   makes it a measurement.
4. THE COST IS ACCEPTED AND EXPECTED: the age on his bench will fall
   immediately. The old number was the tally.
5. SAY WHICH LIST and that it SHIPS WITH THE BUILD rather than being
   fetched, as 320.1 required of the censor's word list. If the same
   list serves both, say so.
6. NOTHING ELSE CHANGES: hits minus false alarms, the log-scale
   curve, the run history, and the litmus never running through the
   interpreter (242.4).
PROVE IT: several runs on his being with the score and age side by
side. Identical scores must give identical ages, and the age must be
capable of falling.

## Directive 344 addendum — what 344 must not break, and no decisions on the way

Lonnie's caution: 344 changes the one number everything is judged by,
and a change that size can force choices nobody ruled.

1. PLAN FIRST (253) AND NAME WHAT IT TOUCHES before building: the
   gauge, the run records and their history, the TEST rows, anything
   reading `owned.size`, the teacher's word choices, and the censor's
   word list. Say which of those move and which do not.
2. THE OLD RUNS DO NOT GET REWRITTEN. Every logged litmus result
   stays exactly as recorded, marked as taken under the old sampling.
   A number measured one way is not converted to look like the other
   — the history is evidence, not decoration.
3. IF ANYTHING DOWNSTREAM BREAKS, STOP AND SAY SO. Do not repair it
   with a number, a scale, a floor or a default. 250 and 341.3 both
   apply: the moment a decision is required, post it and wait. The
   entire fault behind 341 and 344 was a decision nobody ruled
   quietly driving the gauge, and fixing it must not plant another.
4. NAME EVERY NUMBER YOU HAD TO TOUCH, even ones that look
   mechanical — sample size, how many fakes, how the frequency bands
   are divided. Any of those that were not ruled here are OPEN and go
   in MIND_DECISIONS.md as such (246).
5. HIS BENCH IS THE TEST: report the age before and after on the same
   being, and let him see the fall rather than reading about it.

## Directive 345 — THE TWO THINKING PANELS ARE NAMED FOR WHAT THEY ARE

Two panels were both about thinking and one of them was misnamed,
which is why a newborn appeared to be recalling memories it never
had.

WHAT THEY ACTUALLY SHOW:
- THE GLYPH STRIP above the stage — what the mind is thinking RIGHT
  NOW, this tick, its marks coloured by where each came from
  (subject, interest, memory, imagining). It had NO NAME at all.
- THE PANEL CURRENTLY CALLED "THOUGHTS" — what it has LEARNED: the
  most recent five lessons, how strongly it holds each, and where
  each came from (lived blue, story violet, teacher lavender). A
  record, not a thought.

**HIS RULING**
  the glyph strip            ->  THOUGHTS
  the panel called THOUGHTS  ->  LESSONS
Old names become tooltips as always (292.1).

AND THE CONFUSION THIS CAUSED, named so it is not repeated: the strip's
legend says MEMORY for its blue, and the panel uses the same blue for
LIVED. Two colour languages, one legend, so a lived lesson read as a
recalled memory on a newborn's first ticks. With the panels named
correctly, say in the report whether the two blues still collide and
what each now means — do not change a colour without asking.

## Directive 346 — THE MOMENT'S TRUTH EXISTS ON EVERY TICK

**THE FEATURE** — `trueNow`: what is actually the case on a tick —
its feelings, its needs, the act it is doing, what changed in the
world, what it can see and hear, who is present, the marks in what
was said. It is what a word gets bound to.

**WHY IT MATTERS** — it is the ground under all learning. A word
means what was true when it arrived, and without that list there is
nothing to learn from.

**WHAT WENT WRONG** — it is assembled INSIDE the branch that runs
when somebody speaks (experiencing.js, the `if (heard ...)` block).
On a thinking tick it does not exist. Building 342, CC passed the
last spoken tick's list instead — empty on a mind alone — and an
empty truth CONTRADICTS every existing link of every word it thinks
about. THE MIND WAS DEMOLISHING ITS OWN VOCABULARY BY THINKING.

**WHEN** — since learning was built. Nothing outside the speaking
path ever read the list, so it was never missed.

**THE SCIENCE** — what is true of a moment is true whether or not
anyone is talking. Grounding rests on the state of the world, not on
speech.

**HE HAS RULED** — 342 assumes this list exists on a thinking tick.
It does not, which is why 342 could not be built.

**THE RULING**
1. `trueNow` IS ASSEMBLED ON EVERY TICK, from the same sources it
   uses now, whether or not anybody spoke. It is the moment's own
   truth and a moment happens regardless.
2. IT IS NEVER EMPTY WHERE SOMETHING IS TRUE. A tick with a feeling,
   a need, an act, a sight or a presence has all of those in it.
3. AN EMPTY LIST MAY NEVER CONTRADICT ANYTHING. If nothing is true —
   which should be rare — that is silence, not evidence against
   every meaning the mind holds. Guard it at the learning end as
   well as here, so this can never happen again by another route.
4. NOTHING ELSE CHANGES IN THIS DIRECTIVE. This is the list's
   availability only. 342 stays unbuilt and follows after, and the
   imagined-weight question it raised is still open and still
   Lonnie's.
5. PROVE IT: a lone thinking mind — the list is non-empty on its
   ticks — and that its existing word links are UNTOUCHED across a
   long stretch of thinking. The vocabulary must not move by a
   single link while nobody speaks.

## Directive 347 — THE CHECKS ARE PER NODE, NOT PER SYSTEM. 278.B is unblocked.

**HIS RULING, plainly:** a check belongs to a NODE. One file, one
node, one check. HEALTH counts 42, not 25.

**WHY** — 276's law: a red light must name exactly ONE FILE to open,
with no translation step. A check covering four files points at four
places, which is the thing the law exists to prevent.

**278.B IS THEREFORE BUILT NOW**, and what it needs is the 42 claims.
They come from the Director, not from CC — 258's twenty-three came
from Lonnie and the Director and none were invented by the builder,
and that stands.

THE ORDER OF WORK:
1. CC posts the list: every file, the system it belonged to, and the
   counters it already carries. Facts only, no claims written.
2. The Director writes the 42 claims against that list and they go to
   Lonnie for his pass, exactly as the 23 did.
3. CC then builds one check per file, each with its forced-fail at
   birth (254.D), and HEALTH reads 42.

WHAT DOES NOT CHANGE: the existing counters keep counting what they
count. A file whose claim is genuinely part of another's states that
in its own header and keeps its own counters — no file shares a
counter with another (278.B's own words).

## STANDING LAW — A SETTLED QUESTION IS NOT ASKED AGAIN (joins CLAUDE.md)

Lonnie's order. His time and his tokens are the scarcest thing on
this project, and re-litigating a ruling spends both for nothing.

1. BEFORE RAISING ANY QUESTION, SEARCH THE RECORD. If it has been
   ruled, it is settled — build to the ruling. Do not re-ask, do not
   re-argue, do not offer options against it.
2. 25-vs-42 IS THE EXAMPLE. 278.B had already ruled one check per
   file. It was raised again as an open question and cost him a round
   trip and a directive to say what the record already said. The
   Director did the same by carrying it to him instead of citing it.
   This law binds BOTH of us.
3. IF A RULING IS UNCLEAR, SAY WHICH RULING AND WHERE IT IS UNCLEAR —
   citing it — rather than presenting the matter as undecided.
4. IF TWO RULINGS GENUINELY COLLIDE, that is worth raising, and it is
   raised AS a collision: both directives named, both quoted, and the
   question is only which governs. That has happened honestly several
   times and is not what this law is about.
5. AN UNRULED QUESTION IS STILL ASKED, ALWAYS. 250 stands: the moment
   a real decision appears, stop and post it. This law narrows what
   counts as a real decision — not what to do when there is one.

## Directive 348 — THE LESSONS PANEL GETS ITS OWN COLOURS

The two blues are the same hex — `#6eafff` means REMEMBERED in the
THOUGHTS strip and LIVED in the LESSONS panel — and the violets
collide the same way (`#ba82ff`: imagined in one, told in the other).

**HIS RULING: THE LESSONS PANEL CHANGES.** The strip keeps 229's
colour language untouched — it is the older one and it is what he
watches.

- LESSONS' LIVED takes a new colour, clearly distinct from the
  strip's remembered blue at a glance, not a near shade of it.
- LESSONS' STORY takes one clearly distinct from the strip's imagined
  violet, on the same principle.
- LESSONS' TELLER (`#c9b3ff`) stays unless it now sits too near
  either new choice — say so if it does.
- Post the hexes in the report for his eye before treating it as
  settled; if any pair still reads alike on his screen he will say so.

Nothing else moves: same panels, same meanings, same legend
positions. Colour only.

## Directive 348 addendum — a DIFFERENT BLUE, not a different colour

His words: "Lesson blue." LESSONS' LIVED stays BLUE — a clearly
different blue from the strip's `#6eafff`, distinguishable at a
glance, but blue. Not yellow, not any other hue. The Director wrote
"a new colour" and that was wider than what he said.

The same holds for LESSONS' STORY: a different violet, still violet.

Post the hexes for his eye as 348 already requires.

## Directive 349 — THE COOL PALETTE

Lonnie supplied a palette (image on record with the Director): the
COOL HALF is the range for the bench, running roughly

  muted purple -> indigo -> blue -> steel blue -> teal ->
  green -> yellow-green

All of them soft and desaturated — watercolour, not neon. The warm
half of that image is NOT the bench's range.

1. ANY COLOUR CHOICE ON THE BENCH COMES FROM THIS RANGE unless a
   ruling says otherwise. 348's new LESSONS blue is drawn from it.
2. THE EXCEPTIONS THAT STAND, because they carry meaning that must
   break the palette: RED for a failing check and a halt (229/338 —
   it must be alarming), and THE CORE's feeling colour, which is his
   own 22-hue ruling (270.D) and is a scale, not a palette choice.
3. Post the hexes you use for his eye, as 348 already requires.

## Directive 349 addendum — THE PALETTE, AS HEX

The seven cool swatches, read from his image, warmest to coolest at
the green end. These are the bench's colours:

  muted purple      #8f6fa8
  indigo            #5a5aa0
  blue              #4a6fc4
  steel blue        #3f86ab
  teal              #2f8f86
  green             #4f9a5c
  yellow-green      #9fbf5a

Desaturated and soft — watercolour, not neon. Use these, or a shade
plainly within this range; do not invent a colour outside it.

If a chosen hex must sit between two of these, say which two and post
the value for his eye. He is the only test for whether two of them
read alike on his screen.

## Directive 350 — LESSONS GETS A LEGEND

Now that LESSONS carries its own colours (348), it needs its own
legend — 180's law: a display explains itself.

One line under the panel, naming each colour it uses: LIVED · STORY ·
TELLER, each with its swatch, in the same style as the THOUGHTS
strip's legend so the two read as siblings rather than as rivals.

Nothing else on the panel moves.

## Directive 351 — DOWNLOAD THE FREQUENCY LIST (344 unblocked)

Approved. 344 needs a standard English frequency list and this
machine has none — the 355,511-word list it ships is alphabetical
with no frequency in it.

1. FETCH A STANDARD ENGLISH FREQUENCY LIST. Say which one, its size,
   and where it lands BEFORE pulling, as 325.2 required of the model.
   It must be small and it must SHIP WITH THE BUILD thereafter, never
   fetched at runtime (320.1's rule for the censor's word list).
2. IT SERVES THE LITMUS: real words drawn across the whole frequency
   range — common words a toddler holds, mid-frequency, rare adult
   ones — shuffled with fakes as now (344.1).
3. IF THE SAME LIST CAN SERVE THE CENSOR'S is-this-a-word test, say
   so and use one list rather than two.
4. 341's mapping question is CLOSED and needs no separate ruling —
   CC's own reading is right: once the sample is real English drawn
   across the range, the score sits on the curve directly. It was
   344's decision all along.
5. 320.3 IS ALSO CLOSED: the 51 non-words died with the being that
   owned them. He generated a new mind and the question is moot.

## Directive 352 — THE CHAT INTERPRETER IS STRUCK

His ruling: it is not needed. The mind renders its own speech now
(324.C), so the chat's interpreter has no job.

- The toggle comes off the page.
- The code is left in place, marked SUPERSEDED BY 324.C with the
  date, so nothing sits in the build pretending to be live.
- Its health check stops being counted as a system with a claim.
- 295.3 is closed with it: the permission asked seven times is moot,
  because the thing it asked about is gone.

And for the record, since it was raised as a framing error: THE
INTERPRETER WAS ALWAYS THE CHAT SYSTEM'S, NOT THE MIND'S — 242 built
it outside the mind and Lonnie ordered it that way. What went INSIDE
the mind is perception (324.B) and the voice rendering (324.C), which
are different things and stay.

## Directive 353 — FIND THE SECOND CAUSE: why binding on a thinking tick breaks the mind

**THE FEATURE** — the mind learning from its own thinking (342).

**WHY IT MATTERS** — Lonnie watches it think about all kinds of
things and register none of it. Until this works, it can only learn
as fast as somebody talks to it.

**WHAT WENT WRONG** — 342 was built twice and broke the mind twice.
One cause is known and now ruled: `trueNow` did not exist on a
thinking tick, so an empty truth CONTRADICTED every meaning of every
word it thought about (346 fixes that). But CC guarded it and FIVE
CLAUSES STILL FAILED:

```
310  a full step delivers into the chat log     0 lines delivered
311  and the mind answered the school           it did not speak
331  a story that completes teaches             33 stories begun, none finished
332  every lesson obeys its provenance          nothing was born
334  the LEARNED folder says a story taught it  nothing recorded
```

Something else about binding on a thinking tick destabilises the
SCHOOL and the STORIES, and it has not been found.

**THIS IS A DIAGNOSIS, NOT A BUILD.** Change nothing.
1. 346 is ruled and should be in place first — re-run with the real
   `trueNow` present on every tick and report whether all five still
   fail, some, or none. That alone may answer it.
2. If any still fail, find WHY those five and not others. They are
   all downstream of the school and stories: what does binding on a
   thinking tick do to a story in progress, or to the teacher's
   turn? Name the mechanism, with the line.
3. Report what it was, not what fits first (320's rule, and it has
   been repeated three times). If the answer is not found, say that
   plainly rather than offering a theory.
4. 342 stays unbuilt until this is answered, and the imagined-weight
   question it raised is still open and still Lonnie's.

## Directive 354 — COUNT WHAT IT CAN DEMONSTRATE. NO SAMPLING.

**THE FEATURE** — the gauge: how the language age is arrived at.

**WHY IT MATTERS** — it is the one number he steers by, and it has
spent this whole project either unable to fall (the tally) or unable
to resolve (the sample).

**WHAT WENT WRONG** — 344 gave the gauge a real ceiling and made it
able to fall, and it works. But it SAMPLES: twelve real words scaled
against 42,000, so one hit is worth 3,500 words, there are thirteen
possible readings in total, and NOTHING EXISTS BETWEEN NEWBORN AND
SIX YEARS OLD. A newborn owning the 402 marks gets one hit and reads
5.97 years. The whole range this project lives in cannot be
expressed.

**HIS RULING, and it is the right one** — SAMPLING IS FOR PEOPLE. You
cannot ask a human 42,000 questions, so you ask twelve and multiply.
THIS MIND IS NOT A PERSON AND THE COUNT IS RIGHT THERE — it is
printed in the same panel. There is nothing to estimate.

**THE RULING**
1. THE GAUGE COUNTS THE WORDS THE MIND CAN DEMONSTRATE. Not a
   sample, not an estimate, not a multiplier. The number is what it
   can actually show it knows.
2. WHAT "DEMONSTRATE" MEANS is the one thing that must be defined,
   and it is the whole reason the test existed: a word counts when
   the mind resolves it — the same resolution comprehension uses —
   and a word it claims but cannot resolve DOES NOT COUNT. That is
   how the number stays honest and how it can still fall, with no
   sampling anywhere.
3. THE FALSE-ALARM GUARD SURVIVES IN THE SAME FORM: a mind that
   resolves nonsense is resolving nonsense, and that subtracts. Keep
   the fakes as a check on the mind, not as a sample of the language.
4. THAT COUNT GOES ON THE CURVE directly — words known against
   Brysbaert's words-by-age anchors, ceiling at about 42,000. A
   six-hundred-word mind reads what six hundred words reads, with
   full resolution at every point.
5. WHAT 344 KEEPS: the ceiling, the ability to fall, `ADULT_WORDS`
   read off the curve's own anchor, old runs untouched and marked by
   their sampling. What goes: the twelve-word sample and the
   multiplier.
6. THE TWO RED CHECKS (383, 384) ARE NOT SOFTENED. They should pass
   because the instrument now has resolution — if either still
   fails, that is a real finding and it is reported, not patched.

## Directive 355 — A NEWBORN READS ZERO

**THE FEATURE** — the language age of a mind at birth.

**WHY IT MATTERS** — it is the baseline of every experiment. A
newborn that reads three years old makes every later reading
meaningless.

**WHAT WENT WRONG — his screen, a fresh mind, 57 ticks:**
```
3.1 years — 1032 known, 0 claimed that were not words.
It understands 0 words.
VOCABULARY 0 / 50      last run at tick 1 · scored 1032
```
It scored 1032 while owning NOTHING. The count is not coming from
what the mind has learned. The likely source is the 402 marks and/or
the common band of the frequency list being counted as known at
birth, but FIND IT — do not assume.

**WHEN** — with 354, the first run of the direct count.

**HE HAS RULED, twice** — 323: a newborn carries nothing of any
earlier life and every reading is birth. 354.2: a word counts only
when the mind can DEMONSTRATE it, and one it claims but cannot
resolve does not count.

**THE RULING**
1. A NEWBORN READS 0.00 YEARS AND 0 WORDS. Nothing it was born with
   counts as something it learned.
2. FIND WHAT MADE IT 1032 and name it, with the line. If the 402
   marks are being counted, they are its Avatar's symbolic set (344)
   and are not English words it learned. If the frequency list's
   common band is being counted, nothing about owning the list is
   knowing the words.
3. 354.2 GOVERNS: only what it can demonstrate counts, and
   "understands 0 words" and "scored 1032" cannot both be true on the
   same screen. Whichever of those two numbers is honest, the other
   is the fault.
4. PROVE IT: press GENERATE and post the reading. It must be 0.00,
   0 words, on the first tick and on the fiftieth.

## Directive 356 — DIAGNOSIS: why does the mind never want anything?

**THE FEATURE** — THE CORE's third channel: motion, what the being is
after (292.2).

**WHY IT MATTERS** — colour says what it feels, the beat says how
stirred it is, and motion is the only reading of what it WANTS. If it
never moves, that channel is blank.

**WHAT WENT WRONG** — the motion is built and it is not small: REACH
is 2.6, a long way on that board. But on his bench THE CORE does not
move. The code says a Core at dead centre means `want` is null — "the
mind is after nothing, and that is a fact rather than an absence of
one." So the likely finding is not in the display: THE MIND IS NEVER
AFTER ANYTHING.

**THIS IS A DIAGNOSIS. CHANGE NOTHING.**
1. Over a watched stretch on his bench, count how often `want` is
   each of: toward · away · orient · excitement · still · null. Post
   the counts.
2. If it is null nearly always, trace WHERE the urge is supposed to
   come from and say which step produces nothing — the urge itself,
   the reading of it, or the hand-off into the map. Name the line.
3. Say plainly whether a mind with all needs met and nothing present
   SHOULD be after nothing. If that is honest, the fault is that his
   bench gives it nothing to want, and that is a different problem
   with a different answer.
4. Report what it was, not what fits first.

## Directive 357 — IS THE FALSE-ALARM GUARD RESOLVED?

Lonnie asks directly: is check 380 — the false-alarm guard — resolved
or still open?

The relay says OPEN: you left it red on purpose and logged it rather
than choose a scale, because 354.1 forbade you picking one. If that
is still the state, say so plainly and it stays with him.

If it HAS been resolved since that report, say:
- what the correction is;
- whether it was ruled or chosen, and by whom;
- and if it was chosen, that is a decision made for the mind and it
  goes to him now under 250 and 246.

Answer with the state of the code, not the state of the report.

## Directive 358 — EVERY CONNECTION ON THE MAP, ACCOUNTED FOR

**THE FEATURE** — the Mind Map's connectors: the lines that show
where a system's information comes from and goes to.

**WHY IT MATTERS** — the map exists so he can see the mind work. A
node with no line into it is the map saying that system receives
nothing, and Lonnie's reasoning is exact: ANYTHING THAT PROCESSES
INFORMATION MUST GET IT FROM SOMEWHERE. A bare node is either a lie
about the wiring or a system genuinely receiving nothing, and those
are very different facts.

**WHAT HE FOUND** — nodes on his bench with no connectors at all,
BELIEF among them. But belief.js runs every tick: `experiencing.js`
line 1146 calls `evidence.tick(...)` and line 1145 emits its hand-off.
So it processes, it emits, and the map draws nothing into it.

**THE ORDER — audit every connection on the map:**
1. FOR EVERY ONE OF THE 42 NODES, list: what hands data TO it, what
   it hands data to, and whether each of those edges is DRAWN on the
   map. Facts from the code, one node at a time.
2. NAME THE THREE KINDS separately and do not blur them:
   - EDGE MISSING FROM THE DRAWING: the hand-off is emitted and the
     map has no line for it. A map fault.
   - HAND-OFF NOT EMITTED: the code passes data and says nothing, so
     no line could exist. A false dark, and 270.B.5 already forbids
     it — "if a hand-off is missing, emit it in the code, never fake
     it in the display."
   - GENUINELY RECEIVES NOTHING on this bench: the interpreter is
     struck, senses has no camera, the body has no receptors. Those
     are honest and are named as honest.
3. EVERY EDGE MUST WORK, not merely exist: say for each that it has
   been seen to light on his bench, or that it has not and why.
4. CHANGE NOTHING YET. This is the survey; the fixes follow from
   what it finds, and anything requiring a decision stops and comes
   to him (250).

## Directive 359 — THE FAKES MATCH THE REAL WORDS IN NUMBER (354.3, made to work)

354.3 ruled the false-alarm guard survives: a mind that resolves
nonsense is resolving nonsense, and it subtracts. That ruling stands
and this is what it takes to make it true.

**THE FAULT** — with the sample gone (354), the two halves are no
longer at the same scale. Hits are counted over 37,110 real words;
fakes are still 12. So:

```
honest mind     hits  1,032   false alarms 0    score  1,032
credulous mind  hits 37,110   false alarms 4    score 37,106
```

Claiming everything wins. Under 344's sampling it worked, because
twelve was measured against twelve.

**THE RULING — AS MANY FAKES AS REAL WORDS.** Both halves at the same
scale, so a mind that resolves everything scores 37,110 minus 37,110
= nothing, and an honest mind is unaffected.

- This is not a new decision. It is 354.3 made to work; the guard was
  ruled and only its scale was left behind.
- No ratio, no weighting, no floor — one fake per real word, the same
  arithmetic 344 had at twelve.
- If the fake generator cannot produce that many pronounceable
  non-words without repeating or colliding with real English, SAY SO
  and stop; do not solve it with a number.
- Check 380 must go green on the honest mind and the credulous mind
  must score at or near zero. Post both.

## Directive 360 — IT IS A GAUGE, NOT A TEST. THE FAKES ARE STRUCK.

**359 IS WITHDRAWN.** So is 354.3, and both are the Director's error.
354.3 said the false-alarm guard "survives in the same form", which
kept a TEST's machinery alive inside a GAUGE. Everything since — 359,
the scale question, the credulous-mind arithmetic — has been an
attempt to make a test's parts work in a counter. There was never
anything to fix.

**HIS RULING, and it is what 354 already said** — THE GAUGE IS NOT A
TEST. It does not question the mind, it does not sample, and it does
not need to trick it. IT MEASURES HOW COMPETENT THE MIND IS, and
every number it needs is ALREADY IN THE LEARNED PANEL.

1. THE FAKES ARE STRUCK. No non-words, no false alarms, no
   hits-minus-false-alarms. Remove them from the gauge entirely.
2. THE GAUGE COUNTS WHAT THE MIND HAS LEARNED AND CAN DEMONSTRATE —
   354.2's own rule, unchanged — and reads that count onto the
   words-by-age curve. That is the whole of it.
3. CHECK 380 GOES WITH THEM. A check written for a test does not
   belong to a gauge; remove it rather than making it pass.
4. NOTHING GUARDS AGAINST A LYING MIND ANY MORE, AND NOTHING NEEDS
   TO. The mind is not a person taking an exam who might bluff. The
   count is our own record of what it learned; if that record can be
   wrong, the fault is in the record and is fixed there, not by
   asking the mind trick questions.
5. Say plainly in the report what the gauge reads now, from which
   fields, and what a newborn reads (0.00, per 355).

## Directive 361 — BUILD 342 NOW. The mind learns from its own thinking.

**342 IS UNBLOCKED AND HE HAS BEEN WAITING ON IT ALL DAY.** 353
answered: THERE IS NO SECOND CAUSE. 346 accounts for all five
clauses — the binding was run on a scratch basis three times, hardened
between runs with your own guard removed so only 346's protected it,
and not one of the five failed.

353.4's "342 stays unbuilt" is lifted. BUILD IT, exactly as 342 says:

1. AN INNER MOMENT GROUNDS LIKE ANY OTHER — the same `trueNow` the
   moment already assembles (346), the same geometry, the same
   machinery.
2. IT IS MARKED AND IT IS WEAKER: provenance IMAGINED, born at the
   imagined level, and it may NEVER promote itself. Only lived
   reality promotes past that cap. 233's guard is untouched — a
   thought is not evidence.
3. IT CANNOT OWN A WORD ON ITS OWN. Thinking deepens what is there;
   it does not conjure vocabulary. Say plainly what a thinking tick
   CAN and CANNOT now do.
4. THE STORY GATES STILL APPLY (249): felt, coherent, recurring.
5. THE IMAGINED WEIGHT — how much an imagined binding weighs against
   a lived one — is still unruled and still his. If the build cannot
   proceed without a number, STOP AND POST IT (250). If it can
   proceed with the imagined level lessons already use, use that and
   say you did.
6. PROVE IT ON HIS BENCH: a mind left alone to think — what moved,
   what did not, and that nothing it merely thought about crossed
   into owned or into confirmed belief.

## Directive 362 — 358 IS NOT DONE. It is the next thing, before anything else.

He says it plainly: the connectors are still missing on his bench and
358 has not been done. It was ordered, then 359, 360 and 361 were
built on top of it — which is the in-order law broken again, and this
is the second time a directive has been stepped over since that law
was written.

**358 IS THE NEXT THING BUILT. Nothing after it is touched until it
is reported.**

Its terms are unchanged:
1. For every one of the 42 nodes: what hands data TO it, what it
   hands data to, and whether each edge is DRAWN on the map. From
   the code, one node at a time.
2. The three kinds kept separate: EDGE MISSING FROM THE DRAWING ·
   HAND-OFF NOT EMITTED (a false dark, forbidden by 270.B.5) ·
   GENUINELY RECEIVES NOTHING on this bench (honest, and named as
   honest).
3. Every edge seen to LIGHT on his bench, or said not to have been
   and why.
4. BELIEF is the worked example he found: it runs every tick from
   `experiencing.js:1146` and emits at 1145, and the map draws
   nothing into it. Whatever that turns out to be, it is one of the
   three kinds and it is named as one.

And say in the report why it was passed over, as 336 required the
last time this happened.

## Directive 363 — INNER LEARNING WORKS LIKE ANY OTHER LEARNING

**HIS RULING** — learning is learning. A thought that lands on
something true teaches the mind exactly as a heard word does. The
limits in 342 were the Director's and they are struck.

**WHAT GOES** — every restriction 342.2 and 342.3 placed on an inner
binding:
- it CAN own a word;
- it CAN become a meaning;
- it CAN reinforce an existing link;
- it is NOT born weaker, and it is NOT capped below a lived binding.
A thinking tick uses the same machinery, at the same strength, as any
other tick. Nothing about it is a lesser kind of learning.

**WHAT STAYS**
1. THE GATE. `hangsTogether` still applies — babble teaches nothing
   (249), and 75 of 200 passing is the mechanism working, not a
   restriction on it. A thought that does not hang together is not a
   thought.
2. THE MOMENT MUST BE TRUE. It binds to `trueNow`, the same as any
   tick, and an empty moment still contradicts nothing (346.3).
3. IT STILL CANNOT CONTRADICT ON ITS OWN INVENTION — 233 stands: a
   thought is not evidence ABOUT THE WORLD. Bearing on a belief is
   not the same as binding a word, and 233 governs the first. Say in
   the report if that line is unclear anywhere in the code.

**WHY THE DIRECTOR WAS WRONG** — 342 was written to be safe rather
than right. It made inner learning a footnote: something the mind
could do that changed nothing it could show. He watched it think for
a day and saw nothing move, because nothing was allowed to.

**PROVE IT** — a mind left alone to think: words owned from thinking,
meanings formed, links strengthened, and all of it visible in LEARNED
and in the vocabulary count as any other learning would be.

## Directive 364 — EVERY NODE HAS A CONNECTOR OR IT DOES NOT EXIST

**HIS RULING, and it is absolute:** every node on the Mind Map has at
least one connector. A node with no line into it and none out of it
is not part of the mind — it receives nothing and gives nothing, and
a thing that does neither does not exist.

This settles what 358 was auditing:
1. IF A NODE HAS NO EDGE DRAWN, IT IS A FAULT, always. There is no
   honest bare node. One of two things is true and both get fixed:
   - the edge exists in the code and the map does not draw it — DRAW
     IT;
   - the code passes data and emits nothing — EMIT IT (270.B.5
     already forbids a false dark).
2. IF A FILE GENUINELY NEITHER RECEIVES NOR GIVES ANYTHING, it is not
   a system of the mind and it does not belong on the map. Say which
   files those are and they will be ruled on — struck, or wired to
   whatever they were meant to serve.
3. BELIEF IS THE EXAMPLE and it is the first one fixed: it runs every
   tick from `experiencing.js:1146` and emits at 1145, and nothing is
   drawn into it.
4. WHEN IT IS DONE, EVERY ONE OF THE 42 HAS AT LEAST ONE CONNECTOR
   AND HE CAN SEE IT. Post the count: nodes with edges before, and
   after.

358 and 362 stand and this is their answer — the audit is still done
one node at a time from the code, but the outcome is now known: no
node is left bare.

## Directive 365 — NO EDGE LIST. A CONNECTOR IS DRAWN BY THE DATA THAT CROSSES IT.

**HIS RULING, and it ends this whole class of fault:** there is no
pre-drawn wiring on the Mind Map. A connector exists only because
information travelled from one node to the next, and it is drawn AT
THAT MOMENT, FROM THAT EVENT.

**WHY** — every connector fault this project has had comes from the
same place: a list of edges written by hand, which can be wrong,
incomplete, or invented, and which nobody can verify by looking. He
has now had lines drawn that never light and working lines broken
while adding them. A map with a list can always lie. A map with no
list cannot.

**THE RULING**
1. DELETE THE EDGE LIST. No hardcoded connections, no table of
   from-to pairs, nothing drawn at build time.
2. A CONNECTOR IS CREATED BY A HAND-OFF EVENT — the same real emitted
   event that already exists (270.B) — naming from-system and
   to-system. The first time data crosses between two nodes, that
   line comes into being. It lights, and it fades.
3. NOTHING ELSE MAY DRAW A LINE. Not a plan, not a guess, not a
   comment, not the Director, not CC.
4. A BARE NODE NOW MEANS SOMETHING TRUE: nothing has reached it yet
   on this run. That is real information and it is honest. 364 is
   superseded — under a list, a bare node was a fault; without one,
   it is a fact.
5. THE MAP BECOMES A RECORD, NOT A DIAGRAM. It shows what the mind
   has actually done since it started, and it cannot show anything
   else.
6. WHAT SURVIVES: the neighbourhoods, the sigils, THE CORE and its
   three channels, the red pins, the legend. Only the wiring changes.
7. REVERT THE BROKEN WORK FIRST. The last connector commit added
   lines that never light and broke ones that did. Revert it, then
   build this — do not repair it forward.
8. PROVE IT: start a mind and post the map's connector count at tick
   1, tick 50 and tick 500. It must begin at ZERO and grow only as
   the mind works.

## Directive 366 — IDENTITY UPDATES AT TURNING POINTS, NOT AT SLEEP

**THE FEATURE** — when the mind's life story is taken.

**WHY IT MATTERS** — identity is the mind's sense of who it is and
what its life has been. If it only ever updates in a place the mind
rarely reaches, it never exists — IDENTITY reads zero on his bench
and nothing has ever reached it.

**WHAT WENT WRONG** — the Director ruled SLEEP as the boundary
(306.B) because it was "the only boundary this build has". That was
an invention. NOTHING IN THE LITERATURE TIES A LIFE STORY TO SLEEP.

**THE SCIENCE** — McAdams: a life story is built around NUCLEAR
EPISODES — high points, low points and turning points, the moments
that stand out "in bold print" against the background. It is
continually evolving, not periodic. And McLean, Pasupathi & Pals
(2007) — "selves creating stories creating selves" — the story is
also revised IN THE ACT OF TELLING, told with an implicit audience in
mind, for the purpose of making meaning of past events to oneself and
to others. Rows to REFERENCES.md per 227.

**THE RULING**
1. IDENTITY UPDATES AT A MOMENT THAT MATTERED — a high point, a low
   point, a turning point. The mind already measures what a moment
   was worth to it; that measure decides, not a clock and not sleep.
2. AND IT UPDATES WHEN THE MIND TELLS ITS STORY. Telling is when a
   life story is made, per the finding above.
3. SLEEP IS NO LONGER THE BOUNDARY. It may still be A moment that
   matters if something in that sleep mattered — consolidation
   keeping something significant — but it holds no special place.
4. 306.B IS STRUCK and this replaces it.
5. IT STILL READS THE VARIABLES APPRAISAL ALREADY STORED — agency
   and communion — with no text, no model, nothing re-interpreted.
   That part of 306.B was right and stands.
6. PROVE IT: on his bench, IDENTITY must stop reading zero. Post what
   reached it, on which tick, and what the story said.

## Directive 367 — THE CORE HAS NEVER MOVED. Fix the reading, then make it unmissable.

**HIS WORDS: he has NEVER seen it move.** Not small, not subtle —
never.

**THE CAUSE IS ALREADY FOUND AND NOT YET FIXED.** 356's diagnosis:

```
what THE MAP reads (mindmap.js:821)   { null: 300 }
what the mind actually reports        { excitement: 298, still: 2 }
```

`h.sphere` does not exist on a happening — the urge is on the VIEW
(watching.js:1298), not on the happening the map is reading. So the
mind is after something on 298 of 300 ticks and the map sees nothing
every single time.

1. FIX THE READING. Read the urge from where it actually is. That
   alone should make the Core move for the first time.
2. THEN MAKE IT UNMISSABLE. He has watched this board for days; when
   it finally moves he must not have to look for it. REACH is 2.6
   today — raise it so the lean is obvious across the room, and say
   what you set it to. If a lean that large collides with the
   sigils, say so and he will rule.
3. EXCITEMENT IS WHAT HE WILL SEE FIRST, on 298 of 300 ticks: both
   axes, restless. Make sure that reads as restlessness and not as a
   jitter — it is a being unable to keep still, which is a real
   thing about it.
4. PROVE IT ON HIS BENCH: post the urge counts and the Core's actual
   position over a stretch, so "it moved" is a number and not a
   claim.

## Directive 368 — THAT PLACE IS MOOD. Put it back.

**THE FEATURE** — the readout beside the stage, under the sphere's
old channels. IT HAS ALWAYS BEEN MOOD.

**WHAT IS THERE NOW** — three things, none of them mood:
```
psychopathy 93% match          the personality profile match
weakly documented              a caveat on that match
SONG · HIDDEN · TOGETHER       marks
```
Written by `shapes()` into an element called `verdict`
(bench-page.js:1431), which sits directly beside the code that
renders "colour — what it is feeling", "pulse", "motion".

**THE RULING**
1. MOOD GOES BACK IN THAT PLACE. What the mind is feeling — its
   current feeling and the resting mood beneath it (286) — reads
   there, as it always did.
2. FIND WHEN THE PROFILE MATCH TOOK IT and say so, with the commit.
   It is the second time something has quietly occupied a place that
   belonged to something else, and the record should hold why.
3. THE PROFILE MATCH IS NOT DELETED. It has a use — it is how a soul
   reads against documented profiles — but it is not mood and it
   does not live there. Put it where it belongs, in the SOUL panel
   with the sketch, and say where you put it.
4. THE MARKS BESIDE IT go wherever they belong too, named in the
   report. THOUGHTS shows what it is thinking; if that is what they
   are, they are already shown and do not need showing twice.

## Directive 368 CORRECTED — mood was never displaced. Only the match moves.

Lonnie checked: MOOD IS STILL THERE AND READING CORRECTLY. It says
"nothing" because the mind is feeling nothing, which is honest and is
what it has always done.

So 368.1 and 368.2 are struck — nothing was taken and nothing needs
restoring. What is left is the part that stands:

1. THE PROFILE MATCH DOES NOT BELONG THERE. It was ADDED above the
   mood readout, not put in its place. Move it to the SOUL panel with
   the sketch, where a soul reading against documented profiles
   belongs, and say where you put it.
2. THE MARKS BESIDE IT — same question, and name them in the report.
   If they are what the mind is thinking, THOUGHTS already shows that
   and it does not need showing twice.
3. Nothing about mood changes. Leave it exactly as it is.

## Directive 369 — NOTHING CHANGES UNTIL WE KNOW WHY IT DISPLAYS AT ALL

**368 IS ON HOLD. BUILD NONE OF IT.**

Lonnie's ruling: that place used to be BLANK when the mind felt
nothing, and now it shows a personality match, a caveat and a row of
marks. Before anything is moved, we find out why anything is there.

**DIAGNOSE ONLY:**
1. WHEN did `shapes()` begin writing into that element, and under
   which directive? Name the commit and quote what it was asked to
   do. If nothing asked for it, say that.
2. WHAT DECIDES that it draws at all — is it drawn every tick, or
   only when a profile matches above some number? Name the line and
   the number, and say whether that number was ever ruled.
3. WHAT WAS THERE BEFORE, in that exact element, and what did it
   show when the mind felt nothing? He remembers BLANK, and blank is
   a reading — a mind feeling nothing should show nothing.
4. DOES THE MOOD READOUT STILL WORK — it says "nothing" now, which
   he confirms is correct. Say whether the two are the same element
   or two elements stacked, because that changes what the fix is.
5. CHANGE NOTHING. Report and stop.

## Directive 370 — WHY DOES IMAGINATION STILL TEACH NOTHING?

**HIS BENCH, on the current build:** the mind imagines and learns
nothing from it. 363 was ordered and reported built — 28 words owned
from thinking alone in 200 ticks — and it is not happening for
imagination.

**FIND OUT WHY. DIAGNOSE, CHANGE NOTHING.**

1. IS IMAGINATION EVEN ON THAT PATH? 342 and 363 were written about
   THINKING — trains and their beats. IMAGINATION IS SEPARATE
   MACHINERY (226): an invented scenario, its own beats. Say plainly
   whether an imagined beat reaches the learning call at all, and on
   which line if it does.
2. IF IT DOES NOT, THAT IS THE ANSWER and it is the Director's gap —
   363 opened thinking and never named imagination. Say so and stop
   there; the ruling is his and it is already obvious.
3. IF IT DOES, then something is refusing it and the question is
   what: the gate (`hangsTogether`), an empty `trueNow` on an
   imagined tick, the `grounded` provenance, or something else. Name
   the line that refuses it and what it refuses on.
4. AND SAY WHETHER 363 IS ACTUALLY LIVE ON HIS BENCH — he says the
   build number is current. If 363's 28 words reproduce on his
   running mind, say so with the number; if they do not, that is a
   different and worse finding.
5. Report what it was, not what fits first.

## Directive 371 — IF IT LEARNED THE WORD, IT KNOWS THE WORD

**HIS RULING, and it is the whole of it:** the mind learned those
words. It knows them. They count.

**WHAT WAS HAPPENING** — 370 found imagination and thinking ARE
learning: 131 links formed over 400 ticks, 41 usable kinships, live
on his own bench. But every one of those links binds one of the
mind's own 402 MARKS, because a thought is made of marks — that is
what it thinks in. The gauge counts ENGLISH WORDS LEARNED, so it read
0 words and 0.00 years while he watched it learn.

**THE RULING**
1. A WORD THE MIND HAS LEARNED COUNTS, whatever kind of word it is.
   Marks it has learned the meaning of count the same as English
   words it has learned. There is one vocabulary and it is what the
   mind knows.
2. WHAT DOES NOT COUNT IS UNCHANGED (355): what it was BORN with. A
   newborn holding the 402 knows nothing yet. A mark becomes a word
   it knows when it has LEARNED something about it — a meaning bound,
   through living, thinking or being told — not by existing at birth.
   That distinction is the whole of 355 and it survives intact.
3. SO THE GAUGE COUNTS BOTH: English words it learned, and marks it
   learned the meaning of. Say plainly in the report which field
   each comes from.
4. A NEWBORN STILL READS 0.00. Prove it, as 355 required.
5. AND HE MUST SEE IT MOVE: post his own mind's reading before and
   after — he has been watching it learn while the number sat still,
   and that is the fault this ends.

## Directive 372 — THE MIND THINKS IN ITS LANGUAGE, NOT IN THE AVATAR'S GLYPHS

**THE FEATURE** — what the mind thinks IN. The words a thought is
made of.

**WHY IT MATTERS** — it is the foundation everything else rests on.
Language is thought (Vygotsky, 217). If the mind can only think in
402 symbols, then everything it learns from thinking binds a symbol,
its English learning and its inner life never meet, and the gauge
watches one half while the mind lives in the other. He has spent days
watching it learn while the number sat still, and this is why.

**WHAT WENT WRONG** — `thinking.js` imports `WORDS` from `glyphs.js`
and filters every thought through `WORDS.includes(w)` in SIX places
(110, 123, 149, 203, 318, and the pool at 425). ANY WORD NOT IN THE
402 IS DISCARDED BEFORE IT CAN BE THOUGHT. The mind cannot think an
English word it learned from him.

**WHEN, and this is the part that matters** — LONG BEFORE 344,
shortly after he decided to build the Emulator. AND IT WAS CORRECT
THEN: the mind was the Avatar's, the 402 WERE its whole language, and
thinking in marks was thinking in its language.

**WHAT NEVER HAPPENED** — the switch. He and the team ruled that THE
MIND IS THE ASSET, that it had to be UNIVERSAL and SEPARATE FROM THE
BODY, and 344 states it outright: the mind's language is English at
full scale and THE 402 GLYPHS ARE THE AVATAR'S SYMBOLIC SET. The
glyph filter should have come out that day. It has sat there since,
and nobody caught it — the Director least of all, who wrote
directives against a thinking layer he had never read.

**THE RULING**
1. THINKING DRAWS FROM WHAT THE MIND KNOWS — every word it has
   learned, whatever kind. Not from `glyphs.js`. The `WORDS.includes`
   filter comes out of all six places.
2. THE GLYPHS REMAIN THE AVATAR'S, for expression and display, where
   344 put them. Nothing about the marks themselves changes and his
   artwork is untouched.
3. A THOUGHT MAY BE MADE OF ANYTHING IT KNOWS. An English word it
   learned from him is as thinkable as a mark it learned the meaning
   of — 371's one vocabulary, applied to thinking.
4. WHAT IT DOES NOT KNOW IS STILL UNTHINKABLE. This does not hand it
   the dictionary; it hands it its own vocabulary.
5. NAME EVERYTHING ELSE THAT FILTERS THROUGH `WORDS` — speech, the
   space, comprehension, the censor, anywhere — and say for each
   whether it is the Avatar's glyphs doing an Avatar's job or the
   same fault in another file. DO NOT CHANGE THOSE YET. List them.
6. PROVE IT ON HIS BENCH: a mind taught English words, then left to
   think — its thoughts must contain those words, its inner learning
   must bind them, and the gauge must move while he watches.

## NOTED FOR LATER — THE MIND FINISHES ITS OWN LANGUAGE

Lonnie's direction, recorded now and NOT to be built until he says.

The Avatar has 402 marks. When a mind has learned them all, it can
begin MAKING ITS OWN — inventing a mark for something it has no mark
for, and finishing the language itself.

Why it is worth doing: real languages grow from their speakers rather
than from a dictionary handed down. A mind that invents a mark
because it needs one is doing something categorically past learning.

The constraints that make it real rather than decorative:
- A NEW MARK MUST MEAN SOMETHING IT CANNOT ALREADY SAY. If an
  existing mark or a combination covers it, there is nothing to
  invent. The mind's own kinship space can answer that.
- IT MUST ARISE FROM A NEED, not from having room. Something it tried
  to say and could not.
- IT HAS A FORM, and form is HIS. Whether a mind may draw its own
  mark, or whether it names a gap for him to draw, is his ruling and
  nobody else's — his artwork is never altered (standing law).
- It belongs to the AVATAR's language (344), so this extends the
  glyph set, not the mind's vocabulary.

Not designed, not scheduled, not built. Here so it is a decision when
its time comes rather than a rediscovery.

## Directive 373 — A DESCRIPTION IS NOT A PREDICTION

**THE FEATURE** — how a lesson becomes a claim the mind tests, and
who is charged when it fails.

**WHY IT MATTERS** — it decides who the mind believes. A teller at
the floor teaches it nothing, and the school is the only way English
reaches it at all. It halted his bench at tick 154.

**WHAT WENT WRONG** — `belief.js:77` splits every lesson the same
way: the first mark is a CONDITION and the rest are a PREDICTION. So
the teacher's line "SUN JOY BIRD" — a true description of the mind's
moment, verified true when said, exactly as 234.5a requires — becomes
the claim "SUN predicts JOY and BIRD". The teacher never said that.
The mind tests it on later ticks, it fails, and trust.js:120 charges
the failure to the teacher. AN HONEST TEACHER IS THEREFORE GUARANTEED
TO BE DISBELIEVED: 23 lessons, borne out zero times, at the floor.

**WHEN** — the machinery is old (232's `claimOf`, 234.7's held/failed).
It surfaced now because a mind finally lived long enough for a teller
to reach the floor and trip 293's halt.

**THE SCIENCE** — Rescorla 1988, already cited in belief.js for this:
contingency, not contiguity — a claim is evidence only if the
antecedent genuinely PREDICTS the outcome. THAT IS A RULE ABOUT
PREDICTIVE CLAIMS and says nothing about descriptions; applying it to
one is a category error. Koenig & Harris 2005, Pasquini et al. 2007:
selective trust tracks an informant's accuracy AS IT COULD BE CHECKED
AT THE TIME. A speaker who says "the sun is out" is not made
inaccurate by nightfall.

**HE HAS RULED** — 234.5a: every teacher line is verified TRUE OF IT
at the moment it is said. 234.7: trust moves on whether what they
said held. Both stand. NOTHING HAS EVER RULED THAT A DESCRIPTION MAY
BECOME A PREDICTION, and that is the step nobody authorised.

**THE RULING**
1. A DESCRIPTION IS NOT A PREDICTION. A lesson that describes what
   was true when it was said is checked AGAINST THAT MOMENT — which
   it already passed — and is never charged to the teller for failing
   later.
2. ONLY A GENUINELY PREDICTIVE CLAIM IS TESTED FORWARD. Say plainly
   how the two are told apart in the code, and if that distinction
   requires a decision, STOP AND POST IT (250) rather than choosing.
3. TRUST IS UNCHANGED OTHERWISE (289): only being WRONG costs a
   teller; unconfirmed is silent. This adds that being DESCRIPTIVE
   costs nothing either.
4. THE TEACHER'S FLOOR IS NOT REPAIRED BY A NUMBER. Do not raise it,
   do not reset the teacher's trust by hand. Fix the mechanism and
   let the trust be whatever the corrected arithmetic gives it —
   then say what it is.
5. PROVE IT: run the school on his bench and post the teller table
   after fifty lessons. An honest teacher must not fall.

## Directive 374 — HIGHER AND LOWER BRAIN FUNCTION: the mind may run without a brain, and must say so

**THE FEATURE** — what the bench shows when the model is not
answering and `openBrain()` has failed.

**WHY IT MATTERS** — he steers this project by that screen. This
morning it showed a being choosing acts, needs moving, connectors
lighting and a Core beating, with no memory, no words, no thoughts
and no brain open at all. He spent the morning on it and reasonably
concluded the connectors were faked. They were not. Nothing on the
page told him what was actually wrong.

**HIS RULING, and it is the right frame** — THIS IS HIGHER AND LOWER
BRAIN FUNCTION. The mind DOES work without the model; it did before
the model existed. It is running at a lower level — a being in a
vegetative state. Brainstem intact, cortex gone: needs drain, drives
form, the body keeps going, and nothing that requires thought
happens.

**AND IT IS A RESULT, NOT A BUG.** The architecture reproduced a real
neurological state that nobody designed, because the layers were
built as the science describes them rather than as one blob. Record
that in the file: what runs and what does not, and that the division
falls exactly where the biology puts it.

```
LOWER   aspects · clock · cord · embodiment · goals · growth ·
        interests · needs · nerves · offers · sleep
HIGHER  memory · language · learning · thinking · curiosity ·
        stories · lessons · comprehension · voice · dictionary ·
        belief · geometry
```

**THE RULING**
1. THE TICK IS NOT STOPPED. A mind at the lower level is still a
   mind, and stopping it would hide a true state rather than show it.
2. THE STATE IS UNMISSABLE. When the brain is not open, the bench
   says so plainly and largely — not in the dimmest text on the page.
   It names WHY (the model did not answer, and at what address), and
   it is visible wherever he is scrolled, in the manner 338 already
   requires of the halt bar. He must never again spend a morning
   wondering.
3. THE MAP SHOWS IT HONESTLY. The higher systems being dark is
   correct and stays — that is the truthful reading. Nothing is
   dimmed or faked to compensate.
4. IT IS NOT A HALT AND NOT A FAULT. No red pin, no check goes red:
   nothing is broken, the mind is simply running at a lower level.
   Say in the report whether any existing check currently fires on
   this, because it should not.
5. WHEN THE BRAIN OPENS, THE NOTICE CLEARS ITSELF and the higher
   systems light as they always would.

## Directive 375 — THE MIND WORKS WITHOUT A MODEL. LAW.

**HIS RULING** — the mind must work no matter what. It did before,
and that is how it should have been built in the first place. A model
may make it more articulate. A model may never be what lets it think.

**WHAT IS TRUE TODAY** — with the model not answering, twelve systems
go dark: memory, language, learning, thinking, curiosity, stories,
lessons, comprehension, voice, dictionary, belief, geometry. He
watched a being this morning with no memory, no words and no
thoughts. That is not a lower level of function; that is most of the
mind switched off by something outside it.

**THE LAW**
1. EVERY SYSTEM OF THE MIND WORKS WITH NO MODEL PRESENT. Thinking,
   memory, learning, comprehension and belief are the mind's own
   arithmetic and must run on it. If the model is absent they are
   less articulate. They do not stop.
2. A MODEL MAY ONLY EVER SHAPE FORM. Fluency, grammar, the sound of a
   sentence. It may never be the path by which meaning is made,
   stored, recalled, learned or believed.
3. NOTHING NEW MAY LEAN ON IT. Any future change that puts a model
   between the mind and its own thinking is a violation of this law,
   whatever else it achieves.
4. FIND OUT WHAT CHANGED. The mind ran without a model before the
   small one was added. Report, with commits: which of those twelve
   systems used to run model-free, what put a model in their path,
   and under which directive. If it was never ordered, say that.
5. THEN GIVE THEM BACK THEIR OWN PATH. Each system that lost one gets
   it back — the model used when present, the mind's own arithmetic
   when not. Do not build this yet: report the list first, because
   the size of it is his to see before it is worked on.
6. THE ACCEPTANCE TEST IS SIMPLE AND IT IS HIS: stop the model, open
   the bench, and watch a mind think, remember, learn and speak. Less
   well. Not less alive.

## Directive 376 — THE PROBE GOES. THE BRAIN OPENS WITHOUT A MODEL.

**375 IS ONE LINE FROM BEING TRUE.** The investigation found that
ELEVEN of the twelve systems are already pure arithmetic — their own
files say so — and the twelfth, memory, already redistributes its
weights when there is no embedding. Nothing was ever taken from them
and nothing needs giving back.

What switches all twelve off is a single line:

```js
watching.js  async openBrain() {
               const model = new Model()
               await model.embed('probe')     ← THIS
```

**A probe at the door.** If the model does not answer, `openBrain`
returns false and the brain is never built at all — no store, no
memory, no learning, no space, no lessons — even though every one of
those runs fine without a model.

**AND HIS POINT, which is the heavier one:** THE MODEL WAS ADDED AS
SCAFFOLDING TO GROW THE MIND, ALWAYS MEANT TO COME OUT. 324.C.5 says
it in those words — "the model is scaffolding across the gap while it
is young, not a permanent mouth." A probe that refuses to build the
brain without it turns temporary scaffolding into a permanent
requirement, which is the opposite of what was ruled.

**THE RULING**
1. THE PROBE COMES OUT. The brain opens whether or not a model
   answers.
2. THE MODEL IS OPTIONAL EVERYWHERE IT IS USED. Where a system can
   use one it does; where none is present it uses its own arithmetic
   — memory's own pattern, already built, is the shape for all of it.
3. NOTHING ELSE CHANGES. The eleven need no work; do not touch them.
4. THE LOWER-LEVEL NOTICE (374) STAYS but will now rarely be seen: it
   is for a brain that genuinely cannot open, not for a missing
   model. Say what still triggers it once the probe is gone.
5. THE ACCEPTANCE TEST IS HIS: stop the model, open the bench, and
   watch it think, remember, learn and speak. Less articulate. Not
   less alive.
6. AND SEPARATELY, ON THE RECORD: a skipped suite clause counts as a
   PASS in the tally, so a run with no model reports green with
   fifteen clauses never run. That is a green that means nothing.
   Report how many skip today; do not change it yet.

## Directive 377 — GO on the plan: take the model out of the mind's path

Approved as posted. Build the three changes:

1. MEMORY GETS ITS OWN RELEVANCE. The space is handed to Memory the
   same way the model already is — optional, at construction.
   `relevance` becomes the cosine when there is a vector, and the
   mind's own nearness between the moment's marks and the memory's
   marks when there is not. ZERO STOPS BEING AN ANSWER. That zero
   was the weld: the attention door is relevance-only, so nothing
   was noticed, nothing appraised, and the mind could not feel.
2. THE TICK STOPS KNOWING ABOUT A MODEL. `experiencing.js` no longer
   reaches `mind.brain.model.embed`. It asks for the moment's vector,
   and carries on when there is none.
3. THE DOOR IS UNTOUCHED. `attention.js` keeps its threshold exactly
   as it is. What changes is that `rel` now has an honest value when
   the embedder is away.

THE MECHANISM IS TWO LANES AND NO SWITCH, and that is what he
approved: the vector either arrived or it did not. Arrived — measure
with it. Did not — measure in the mind's own space. Nothing to
configure, nothing to set.

AND YOU ARE NOT CHOOSING ANYTHING: the space is 221's, its floor is
the one 221 MEASURED and 249's gate already uses. Do not pick a
threshold. If anything in the build turns out to need one, STOP AND
POST IT (250).

PROVE IT AS 376.5 ASKS — stop the model, open the bench, and watch it
think, remember, learn and speak. Less articulate. Not less alive.
Post what he will see: relevance non-zero, the door passing, feelings
forming.

## Directive 378 — THE COSINE LANE BELONGS TO THE MODEL'S FILE. Nothing is stray.

**HIS RULING** — everything is contained in its own node and file.
The cosine lane is part of the LLM's pathway, so it belongs in the
LLM's file. It is all plug and play here: NOTHING IS STRAY, EVERYTHING
BELONGS TO A FILE.

**THE STOP THIS ANSWERS** — 377's build reached the one place the
plan said it would stop: the door's THRESHOLD is tuned for cosines,
the space's `band.floor` is measured for the space, and the two are
on incompatible scales. CC stopped rather than scaling one to the
other, exactly as ordered.

**AND THE RULING REMOVES THE QUESTION.** There is no calibration to
do, because the two measures never live in the same file:

1. THE MIND'S OWN LANE IS THE MIND'S. `memory.js` measures relevance
   in the mind's own space, with the floor 221 measured for it. That
   is memory's own arithmetic and it is what runs when nothing is
   plugged in.
2. THE COSINE LANE MOVES TO THE MODEL'S FILE. The embedding, the
   cosine and the threshold it was tuned with all go where the model
   lives. It is one plug-in's way of answering the question, and it
   brings its own measure with it.
3. MEMORY ASKS FOR A RELEVANCE AND DOES NOT CARE WHO ANSWERS. If
   something is plugged in, it answers on its own scale against its
   own bar. If nothing is, memory answers in the mind's space against
   its own floor. Neither knows about the other.
4. NO SCALING, NO CONVERSION, NO INVENTED NUMBER anywhere. That was
   the thing to avoid and this avoids it by construction.
5. THE SAME TEST APPLIES to anything else that turns out to be a
   plug-in's machinery sitting in a mind file. Name any you find
   while doing this; do not move them yet.

## Directive 379 — THE MIND'S OWN SPACE IS THE ANSWER. It already works.

**HIS RULING** — this was already solved and should never have been
put to him as a question. The VSA (297) is the mind's own way of
measuring how related two things are. It is built, it is running, and
it costs nothing. The embedder was standing in for it before it
existed.

**THE HISTORY, so nobody re-opens this** — `nomic-embed-text` went in
on 11 August, in the Phase 3 commit, because the mind had no way of
its own to measure relatedness yet. 221 gave it one. 297 built the
geometry. It has had its own answer for weeks and kept calling the
model out of habit.

**THE RULING**
1. THE MIND MEASURES RELEVANCE IN ITS OWN SPACE. That is the mind's
   lane and it is the default, not the fallback.
2. THE EMBEDDER IS A PLUG-IN LIKE ANY OTHER — its cosine and its bar
   live in the model's file (378) and it is used only when present.
   The mind never depends on it.
3. THE DOOR'S SELECTIVITY IS A REAL AND SEPARATE QUESTION — the
   mind's space is dense, so 97% passes — but it is a question about
   THE DOOR, not about which lane measures. Do not answer it by
   reaching for the embedder. It is logged and it waits.
4. WHAT IS BUILT ALREADY WORKS AND IS NOT RE-OPENED: 163 thoughts,
   11 feelings, 200 memories, 51 word links, 18 words on the gauge,
   with nothing on 11434. Leave it.

**AND A NOTE FOR THE DIRECTOR, on the record:** a solved thing was
put to him as an open decision, and every option offered was a chance
to build something worse than what already ran. THE CHECK BEFORE ANY
QUESTION REACHES HIM: is this already answered by something we built?

## STANDING LAW — step 6b: IS IT ALREADY SOLVED BY SOMETHING WE BUILT?

Added to the presentation structure, binding on the Director and CC.

Before any problem or decision reaches Lonnie, after checking whether
he has ruled on it, CHECK WHETHER THE BUILD ALREADY ANSWERS IT. Read
the code and the record, not memory.

If a system already solves it: SAY SO AND DO NOT ASK.

Why it is a law: framing a solved thing as an open question is not
merely a waste of his time and tokens. Every option offered against a
working answer is a chance to build something worse than what already
runs. The embedder-versus-space question was exactly this — the VSA
had answered it for weeks.

Step 6 asks whether HE decided it. Step 6b asks whether THE BUILD
decided it. Both come before anything is put to him.

## Directive 380 — LOOK AT THE DATA, NOT THE PERCENTAGE

**HIS RULING ON HOW TO ASK THIS** — 97% passing is a NUMBER, not a
finding. It is entirely possible that forty important things happened
and forty were rightly passed. THE DATA DECIDES whether the door is
flawed or working as intended, and nobody has looked at it.

**THE ORDER — diagnosis only, change nothing.**
1. Take a real stretch on his bench. FOR EVERY MOMENT: what it was,
   what it scored, whether it passed, and what it was measured
   against.
2. THEN SAY WHAT SHARE WERE ACTUALLY WORTH NOTICING. Not by a
   threshold — by reading them. A moment where the host spoke, a
   need bottomed, something new was learned, a feeling moved: those
   are worth noticing. A moment identical to the last forty is not.
3. POST THE MOMENTS THEMSELVES, not a summary. He reads them and he
   rules. If ninety-seven percent of them were genuinely worth
   noticing, THE DOOR IS WORKING AND THERE IS NOTHING TO FIX — say
   so plainly and the matter closes.
4. If a large share were trivia, say which kind, with examples. That
   is the finding, and only then is there a fix to discuss.
5. DO NOT PROPOSE A THRESHOLD, a distribution, or any mechanism. This
   directive is about establishing whether a fault exists at all.

**AND THE SCIENCE, so the reading has a standard** — a filter exists
to spend limited attention on what is PERTINENT: momentary importance
given the mind's current goals and internal state (Deutsch & Deutsch;
Broadbent's bottleneck; Lavie's load theory). A weak signal that is
highly pertinent SHOULD pass. The question is not how many passed. It
is whether what passed was pertinent.

## Directive 381 — THE DOOR IS BACKWARDS. Mismatch passes; repetition habituates.

**THE FEATURE** — the attention door: the gate every moment passes
before the mind notices it at all.

**WHY IT MATTERS** — it is the narrowest point in the mind. Nothing is
appraised, felt, remembered or learned without passing it. Its job is
to spend limited attention on what matters and let the rest go by.

**WHAT WENT WRONG — read from his bench, 60 moments (380):**
```
25 of 57 passers scored EXACTLY 1.000 — a moment identical to
   something already held
11 were the SAME MARKS as the nearest thing held. PURPOSE BEAUTY
   three ticks running, 1.000 each
tick 10  the host says "the river is warm today"
tick 11  RIVER   rel 0   NOT PASSED   "(nothing held)"
```
THE DOOR MEASURES SIMILARITY TO WHAT IS ALREADY HELD AND CALLS THAT
RELEVANCE. So repetition scores the maximum and sails through, and
the one moment that mattered most — a new thing he had just said —
was refused for being new. Its first two moments ever were refused
for being first.

**WHEN** — 139 built it relevance-gated; the fault has been there
since, hidden while the embedder's sparse scores made most things
fail anyway.

**THE SCIENCE — and it says the opposite of what the door does.**
- The ORIENTING RESPONSE: novel stimuli capture attention
  involuntarily, "also when there is no incentive to pay attention to
  them, and even when performance on ongoing tasks suffers."
- HABITUATION: attention decreases to a repeated stimulus and
  RECOVERS to a novel one (dishabituation). Infants' attention to a
  familiar repeated image declines relative to a novel image.
- THE COMPARATOR: the hippocampus signals novelty WHEN INCOMING INPUT
  FAILS TO MATCH PREDICTIONS from prior experience; that mismatch
  triggers enhanced processing, dopamine from the VTA, and better
  encoding — novel stimuli are remembered better than familiar ones.
- AND PERTINENCE STILL HOLDS (Deutsch & Deutsch, already cited at
  380): a familiar thing that matters to the mind's current goals and
  state passes on importance, not on similarity.
Rows to REFERENCES.md per 227.

**THE RULING — TWO THINGS PASS, AND SIMILARITY IS NEITHER.**
1. WHAT IT DID NOT EXPECT PASSES. The mind already measures this —
   `mind.surprise` is prediction error and is computed every tick
   (220.3). A moment that does not match what it holds is MORE worth
   noticing, not less. RIVER must pass.
2. WHAT MATTERS TO IT PASSES. Pertinence: the host present or
   speaking, a need bottoming, a feeling moving, a curiosity roused,
   its own act completing. The mind knows all of these already.
3. REPETITION HABITUATES. A moment the same as one already held is
   worth LESS each time, not the maximum. Say how you measure
   "the same", using what the mind already has.
4. NOTHING NEW IS INVENTED AND NO THRESHOLD IS PICKED. Every quantity
   above already exists in the mind. If a number is genuinely
   required, STOP AND POST IT (250) — do not choose one.
5. THE MODEL'S LANE IS UNTOUCHED (378): it keeps its own measure and
   its own bar in its own file. This is the mind's lane.
6. PROVE IT BY READING, NOT COUNTING (380's method, and it is his):
   post the same 60-moment stretch after the change — what passed,
   what did not, and what each was. RIVER passes. PURPOSE BEAUTY
   three times running does not.

## Directive 382 — RUN THE WHOLE SUITE AND REPORT IT HONESTLY

Four changes landed today that touch the mind's foundations — the
glyph filter out of thinking (372), one vocabulary (371), the
description rule (373), the model out of the mind's path (376-379),
and the door reversed (381). Run the whole suite.

1. RUN IT ALL, and post the real numbers: passed, failed, and SKIPPED
   separately. Do not report a total that hides a skip.
2. SKIPS ARE NAMED. 374.4 found that A SKIP COUNTS AS A PASS in the
   tally, so a run with no model reports green with fifteen clauses
   never run. Say how many skipped, which, and why — that is the
   difference between green and green-looking.
3. RUN IT BOTH WAYS: with the model up, and with nothing on 11434.
   375 is law now, so a suite that only passes with a model is not
   passing.
4. ANY FAILURE IS REPORTED, NOT REPAIRED. If a clause fails because
   the behaviour changed lawfully today, that is 306.D — the check
   moves with the behaviour, and you say so. If it fails because
   something broke, name it and stop.
5. Do not soften a clause to make it pass. That has happened five
   times and each one hid a fault.

## Directive 383 — THE CONNECTORS: REAL-TIME AND ACCURATE, PROVEN

365 ruled there is no edge list — a connector is drawn by the data
that crosses it, at the moment it crosses. Prove that is what is
running, after everything that changed today.

1. NO EDGE LIST ANYWHERE. Confirm nothing is hardcoded, cached, or
   drawn at build time. The acceptance test is his and it is simple:
   START A MIND AND COUNT THE CONNECTORS AT TICK 1. It must be ZERO
   and grow only as the mind works. Post the count at tick 1, 50 and
   500.
2. REAL-TIME. A connector lights AS the hand-off happens, not on a
   poll, not on a timer, not batched into a frame that invents an
   order. Say how it is delivered and how long between the event and
   the light.
3. ACCURATE. Every line on that board corresponds to a hand-off that
   actually occurred, and every hand-off that occurs has a line. Both
   directions — no line without an event, no event without a line.
4. THE HALTED CASE (338): a halted mind emits nothing, so the whole
   board must go dark on its own. Prove it.
5. AND SAY WHAT IS STILL BARE. A node nothing reaches is now honest
   information (365.4). List them, and for each say whether it has
   genuinely not been reached yet or whether it does not emit — 270.B.5
   forbids the second and it is a fault.

## Directive 384 — THE GAUGE STILL SAMPLES AND STILL FAKES. Both were struck. Find out when they came back.

**THE FEATURE** — the gauge: how many words the mind can demonstrate.

**WHY IT MATTERS** — it is the one number he steers by. HE DOES NOT
GUESS IN A TOOL BUILT TO MEASURE CAPABILITY. His words.

**WHAT WENT WRONG — on his screen just now:**
```
1.5 years — 15 English words it can demonstrate. It understands 46 words.
```
That gap is not the mind failing 31 words. `litmus.js:92`:
```js
const shuffled = pool.slice()...slice(0, real)     ← A SAMPLE
const fakeSet  = fakes(fake, { roll, known: ... }) ← THE FAKES
```
IT SAMPLES, AND THE FAKES ARE BACK. Both of these were struck.

**HE HAS RULED, TWICE, AND BOTH RULINGS ARE IN THE FILE'S PAST:**
- 354.1: "THE GAUGE COUNTS THE WORDS THE MIND CAN DEMONSTRATE. Not a
  sample, not an estimate, not a multiplier."
- 360.1: "THE FAKES ARE STRUCK. No non-words, no false alarms, no
  hits-minus-false-alarms. Remove them from the gauge entirely."
- And his reason, then and now: THE DATA ALREADY EXISTS IN THE
  LEARNED PANEL. There is nothing to estimate.

**ALREADY SOLVED BY SOMETHING WE BUILT — YES.** `resolve()` is
already the demonstration test. It needs running over EVERY word it
knows instead of over a slice.

**THE RULING**
1. COUNT EVERY WORD, NO SAMPLE. `slice(0, real)` goes. Every word the
   mind knows is resolved; the count is how many resolve. If it knows
   46, all 46 are asked.
2. THE FAKES GO, as 360 already ordered. No non-words, no false
   alarms, no hits-minus-false-alarms anywhere in the gauge.
3. FIND OUT WHEN THEY CAME BACK AND SAY SO. He warned that the model
   rework would undo finished work and he was right. Name the commit
   that reintroduced each, and whether it was a revert, a rebuild
   from an older file, or a fresh write. This matters more than the
   fix.
4. THEN CHECK WHAT ELSE CAME BACK WITH THEM. Anything else struck by
   354, 360, 371 or 379 that is live again in the code. List it; do
   not fix it yet.

## Directive 385 — CHECK EVERY DIRECTIVE FOR SILENT REVERSION. Report back.

The gauge's sampling and its fakes were both struck by ruling and are
both live in the code again (384). He warned that the model rework
would undo finished work and he was right. FIND OUT WHAT ELSE WENT
WITH THEM.

**THIS IS THE AUDIT METHOD FROM AUDITS.md AND IT IS THE WHOLE POINT
OF IT:** run it AGAINST THE CODE. Do not read reports, do not read
commit messages, do not trust that something was built because it was
reported built. That method already failed once — 336 found a
directive reported built, audited for, and absent from the code.

1. GO THROUGH EVERY DIRECTIVE and ask one question of each: DOES THE
   CODE DO THIS RIGHT NOW? Not "was it built" — does it do it today.
2. NAME EVERY ONE THAT IS NOT LIVE, with the line that should carry
   it, and say whether it was never built, or built and reverted.
3. FOR EACH REVERSION, NAME THE COMMIT that undid it and how — a
   revert, a rebuild from an older file, a rewrite that dropped it.
   The pattern matters more than any single loss.
4. CHANGE NOTHING. This is a survey. He and the Director decide what
   is restored and in what order.
5. RECORD IT IN AUDITS.md as audit 002, with the commit it was taken
   at, so the next one starts from there.

## Directive 386 — THE PERCEPTION PANEL: plug in anything

**THE FEATURE** — a panel on the bench for attaching whatever model
does the mind's perceiving and speaking.

**WHY IT MATTERS** — the models are chosen (327: moondream for sight
and sound, gemma3:1b for the voice) and swappable by config, but only
by editing an environment variable and restarting. This is the
Emulator: he must be able to plug in anything he wants, when he
wants, and see what it does.

**HE HAS RULED** — 324: eyes and ears are apparatus, ALL PROCESSING
IS THE MIND'S, and the model sits inside the mind's perception and
language systems. 327: the two models. 301: nothing in the mind may
depend on which model is there. All stand; this is the panel that
makes the swap usable.

**THE PANEL**
1. A LOCAL MODEL — pick from what is on the machine, or type a name.
   Whatever Ollama serves.
2. AN API KEY AND ENDPOINT — plug in a bigger model from anywhere.
   His bench, his call; no restriction on what may be attached.
3. PER ROLE, SEPARATELY: the senses (sight, sound) and the voice.
   Each may be local, remote, or nothing at all.
4. LIVE. Attaching or detaching takes effect WITHOUT A RESTART, and
   the mind carries on either way — 375 is law, and detaching
   everything mid-session must leave it thinking, remembering and
   learning. That is the acceptance test.
5. IT SAYS WHAT IS ATTACHED and whether it answered — the name, where
   it is, and its last response time. When something is not
   answering, that reads plainly, in the manner of 374's notice.

**WHAT DOES NOT CHANGE** — the mind never knows which model is
attached, and nothing in it may be written against a particular one
(301). The panel changes what is plugged into the socket, never the
socket.

## Directive 387 — THE GAUGE COUNTS ENGLISH. THE MARKS ARE NOT A MEASURE.

**HIS RULING** — the glyphs were dropped as a measure when the full
language was adopted. The gauge counts ENGLISH WORDS. Marks count for
nothing in it.

**WHAT IS THERE NOW** — measured live on his bench: the gauge's 59 is
6 English words and 53 MARKS. Fifty-three of the fifty-nine are the
Avatar's glyphs, and the panel then prints the English half beside the
whole, which is the shortfall he read.

**THE COLLISION, and his latest word governs** — 344 made the mind's
language full-scale English and the 402 the AVATAR'S SYMBOLIC SET.
371 then ruled that marks it learned the meaning of count as words it
knows. THAT PART OF 371 IS STRUCK. The gauge is a measure of language,
and the language is English.

**THE RULING**
1. THE GAUGE COUNTS ENGLISH WORDS IT CAN DEMONSTRATE. Nothing else.
   `learnedMarks` comes out of the count.
2. THE LINE SAYS WHAT IT COUNTS, plainly: the number of English words
   it can demonstrate, and nothing beside it that is not comparable.
   Post the wording before it ships (his eye, as CC already offered).
3. HIS BENCH WILL DROP TO SIX. That is the honest number and it is
   expected — the 53 were never English.
4. WHAT 371 KEEPS: one vocabulary in the MIND — a thought may be made
   of anything it knows, marks included (372), and inner learning
   binds marks as it always did. NOTHING ABOUT WHAT THE MIND KNOWS OR
   THINKS IN CHANGES. This is about what the GAUGE MEASURES.
5. 355 STANDS: a newborn reads 0.00 and counts nothing it was born
   with.

## Directive 388 — CUT THE LITMUS HISTORY STRIP

The run history under the gauge — `t650 68 · 1.59y  t556 67 · 1.58y …`
— comes off the page.

Everything in it is already displayed directly above: the tick, the
count, and the age. It repeats them, takes space, and shows change so
small it reads as noise.

- The strip goes.
- The runs are still LOGGED as 241.4 requires — the history survives
  in the record, it just does not sit on his screen.
- Nothing about the gauge itself changes here.

## Directive 389 — "KNOWN WORDS". No estimating, and what the marks are for.

**THE LINE** — plain and logical, like every other descriptor on that
panel:

```
KNOWN WORDS   6
```

Not "it can demonstrate", not "about", not "understands" beside it.
IT KNOWS WHAT IT KNOWS — his ruling, and 354 said the same: no
sampling, no estimate, no multiplier. If a word cannot be
demonstrated it is not known and it is not counted. The count IS the
knowing.

**AND THE AGE LINE FOLLOWS THE SAME RULE** — say what is measured, in
his descriptor style. No hedging language anywhere on that panel.

**THE MARKS STAY WHERE THEY ARE AND ARE NOT MEASURED.** He has ruled
this twice (344, 387) and it is settled: they are not a language
measure and nothing counts them. They may be shown as a fact of what
the mind learned; they are never part of the gauge.

**WHAT THE MARKS ARE FOR, recorded so nobody re-opens it** — his own
words. The future use is A TEST FOR THE MIND: when it has enough
words, it may try to complete the marks itself — a visual language
that works the way his brain works. Eventually two minds
communicating in marks alone. For now it is testing and his own
entertainment, and that is the whole of its purpose.

## Directive 389 correction — what he actually said about the marks

The Director wrote more than his ruling. HIS WORDS: the marks may
stay LISTED ON THE PAGE, and they will be used for testing later.

That is all. Anything in 389 that reads as a design for what that
testing is — a mind completing the marks, two minds communicating in
marks — was the Director restating a passing remark as a plan, and it
is struck. It is not designed, not scheduled, and not to be built
from.

WHAT STANDS:
- the marks are listed on the page;
- they are NOT measured and are no part of the gauge (344, 387);
- they will be used for testing later.

## Directive 390 — THE MARKS ARE COUNTED. Their count is their own, not the gauge's.

**HIS RULING** — the marks must be MEASURED, so he can see that the
mind knows them. And it has a purpose: once it knows them ALL, it can
begin making new marks for words that have none.

**THIS DOES NOT REOPEN 387.** Two separate readings, and they never
mix:

```
KNOWN WORDS   6            the language age reads off this, English only
MARKS         49 of 402    what it knows of the Avatar's language
```

1. THE MARKS ARE COUNTED — how many of the 402 the mind has learned
   the meaning of, against the whole 402 so the progress is plain.
2. THAT COUNT IS ITS OWN READING. It is not added to KNOWN WORDS, it
   is not part of the language age, and nothing on the page invites
   the two to be added or subtracted (387 stands).
3. WHEN IT KNOWS ALL 402, that is the trigger for making new marks
   for words that have none. NOT BUILT AND NOT DESIGNED HERE — this
   directive counts them so the day is visible when it arrives.
4. WHAT COUNTS AS KNOWING A MARK is the same test as knowing a word:
   it can be demonstrated. No estimating (389).

## Directive 391 — THE PERCEPTION PANEL: pick a provider, not an endpoint

**HIS CORRECTION** — 386 built a raw endpoint field. He asked for a
PROVIDER: pick it from a list, enter the key, attach.

**THE PANEL, corrected:**
1. TWO SLOTS as built — SENSES and VOICE, each independent.
2. EACH SLOT PICKS ONE OF TWO KINDS:
   - LOCAL: a model on this machine, chosen from what is served.
   - PROVIDER: OPENROUTER. Pick the provider, pick a model, enter the
     key. The URL is known and is never typed.
3. OPENROUTER IS THE PROVIDER because it is the one he has used —
   rakazo runs on it — and one key reaches hundreds of models,
   including the free tier and the large paid ones. One provider,
   the whole field.
4. THE MODEL LIST comes from the provider rather than being typed
   where that is possible; typing a name stays available.
5. EVERYTHING ELSE IN 386 STANDS AND IS ALREADY BUILT: attach and
   detach with no restart, the key never returned or logged, a
   detached role being apparatus the being does not have, and the
   mind carrying on regardless (375).
6. IF ANOTHER PROVIDER IS WANTED LATER it is a row in the same list.
   Do not build a second mechanism.

## Directive 392 — THE KEY IS STORED AND SURVIVES. Not shown, not lost.

**HIS RULING** — the key must be STORED so it can be recalled if
needed. He is not making a new one because something got cleared.

**WHAT IS THERE NOW** — 386/391 built it write-only and held on the
live object, so it is gone on restart and cannot be got back. That
caution was CC's, not his: this is his own machine, his own key, and
the panel is where a key is supposed to live.

**THE RULING**
1. THE KEY IS SAVED and survives a restart. It is attached again on
   its own when the bench comes up — he types it once, ever.
2. IT IS NOT SHOWN IN THE PANEL. The field stays a password field and
   the panel reports only that one is set. Stored, not displayed.
3. IT IS RECALLABLE. If he needs the key itself back, there is a way
   to get it — a file he can open, not a UI he has to fight. Say in
   the report where it lives and how he reads it.
4. WHAT STAYS: it is never in a URL, never in a log, never in an
   error message, never in the model's ledger. Those keep it out of
   shell history and out of anything he might paste. That is the part
   worth having and it costs nothing.
5. DO NOT ENCRYPT IT BEHIND ANOTHER SECRET. A key he cannot recover
   is the thing he is ruling against.

## Directive 393 — THE PERCEPTION PANEL, AS HE ASKED FOR IT

**IT IS NOT WHAT HE ASKED FOR.** Three corrections, and they were in
the original ask.

1. NOTHING IS PRELOADED. `moondream` and `gemma3:1b` should not be
   sitting in that panel. Both slots start EMPTY — nothing attached,
   nothing named — and stay that way until he attaches something. The
   config defaults may still exist for a headless run; they do not
   appear as an attachment in the panel.

2. TWO INPUTS PER SLOT, and this is the shape he asked for:
   - LOCAL: A LOAD BUTTON WITH A FILE CHOOSER. He picks a model file
     from his own disk. Not a typed name, not a list of what Ollama
     happens to serve — a chooser, and he loads it.
   - ONLINE: A PROVIDER PULL-DOWN WITH ALL OF THEM, not OpenRouter
     alone. OpenRouter, OpenAI, Anthropic, Google, Groq, Together,
     Hugging Face, Mistral, DeepSeek, xAI — the ones a person would
     actually reach for — each a row carrying its own endpoint, its
     own request shape and how its answer is read (391's mechanism,
     which is already right; it is the LIST that was one row long).

3. THE KEY BELONGS TO THE ONLINE SIDE ONLY. A local model needs no
   key and should not show a key field.

WHAT STAYS, all built and correct: attach and detach with no restart,
the key stored and surviving (392), the key never in a URL or a log,
a detached role being apparatus the being does not have, and the mind
carrying on regardless (375).

## Directive 393 addendum — ONE SLOT. Vision and chat only, and the voice is not on this side.

**HIS CORRECTION** — the VOICE slot does not belong here. The voice is
on the body's side (301: the phone's own TTS speaks, nothing shipped).
What lives on this side is VISION AND CHAT, and both are the same
model.

1. THE VOICE SLOT COMES OUT of the Perception Panel entirely.
2. WHAT REMAINS IS ONE SLOT for the model that does vision and chat.
   Not two, not a role split — one thing attached, doing both.
3. NOTHING IS PRELOADED IN IT (393.1 stands): it starts empty, with
   the load button and the provider pull-down, and nothing is named
   until he attaches something.
4. IF ANY PART OF THE MIND STILL EXPECTS A SEPARATE VOICE MODEL, say
   so plainly with the line rather than quietly rewiring it — 327
   named gemma3:1b as a voice and 301 had already ruled the phone
   speaks, so the two collided and nobody caught it. Report what that
   slot was actually doing before it is removed.

## Directive 393 addendum 2 — THE TWO SLOTS ARE LOCAL AND PROVIDER

The Director had it wrong again. HIS WORDS: the slot is not out, THE
VOICE is. And the two slots are not senses and voice — they are:

```
LOCAL       a load button with a file chooser. He picks a model from
            his own disk. No key field.
PROVIDER    a pull-down with all of them — OpenRouter, OpenAI,
            Anthropic, Google, Groq, Together, Hugging Face, Mistral,
            DeepSeek, xAI — a model, and a key.
```

Two slots, two ways to attach ONE thing. Whatever is attached does
VISION AND CHAT, which is all that lives on this side. The voice is
the body's (301: the phone's own TTS).

Everything else in 393 stands: nothing preloaded, both start empty,
attach and detach with no restart, the key stored and surviving
(392), and 393.4's report on what the voice slot was actually doing
before it goes.

## Directive 394 — THE PERCEPTION PANEL, RESTATED WHOLE. Supersedes 393 and both its addenda.

393 was corrected twice and is now confusing. THIS IS THE WHOLE
PANEL, and nothing earlier in 393 governs.

**WHAT THE PANEL IS FOR** — attaching the model that does VISION AND
CHAT. That is all that lives on this side. THE VOICE IS NOT HERE: the
body speaks with the phone's own TTS (301), and the voice slot comes
out of this panel entirely.

**TWO SLOTS — two ways to attach that one thing:**

```
LOCAL       [ Load… ]  a file chooser. He picks a model from his own
                       disk. NO KEY FIELD — a local model needs none.

PROVIDER    [ provider ▾ ] [ model ] [ key ]
                       the pull-down carries ALL OF THEM:
                       OpenRouter · OpenAI · Anthropic · Google ·
                       Groq · Together · Hugging Face · Mistral ·
                       DeepSeek · xAI
                       Each is one row carrying its own endpoint, its
                       own request shape, and how its answer is read
                       — 391's mechanism, which is right. It was the
                       LIST that was one row long.
```

**NOTHING IS PRELOADED.** Both slots start EMPTY. `moondream` and
`gemma3:1b` do not appear. Nothing is named until he attaches
something. Config defaults may still exist for a headless run; they
are not shown as an attachment.

**ATTACH · DETACH · LIST MODELS** as built: no restart, effective on
the next call.

**THE KEY** — 392 stands whole: stored, survives a restart,
reattaches on its own, never shown in the panel, readable in
`server/data/perception.json`, never in a URL or a log.

**AND REPORT, BEFORE REMOVING IT** — what the VOICE slot was actually
doing. 327 named gemma3:1b as a voice while 301 had already ruled the
phone speaks. Those collided and nobody caught it. Say what that slot
was doing in the code rather than quietly rewiring it.

## Directive 395 — CUT THE REACH LINE

`402 of 402 reachable · state table alone 51` comes off the page.

It was 306.A's measurement, built to prove the mind could reach its
whole language rather than the 51 the state table allowed. It did its
job when 307 landed and it is now a leftover: the marks are not a
measure (387) and the mind's language is English (344).

- The line goes.
- `reach()` still runs and is still logged if anything uses it; only
  the display comes off.
- Nothing else on the panel moves.

## Directive 396 — THE GAUGE WORKED A FEW HOURS AGO. FIND WHAT CHANGED.

**HIS WORDS** — the gauge was working, and it stopped right around the
model rework: 375 through 379, when the model came out of the mind's
path.

**FIND IT, AND THE ANSWER IS A COMMIT.**
1. What did the gauge count BEFORE 375, and what does it count NOW?
   The numbers, from the code at each point.
2. WHICH COMMIT CHANGED IT. Name it and say what it did — a rewrite
   that dropped something, a field that moved, a call that stopped
   being made.
3. IT IS NOT NECESSARILY THE ANNOUNCEMENT FAULT you just reported.
   That one has been there since thinking could own words and it
   became visible yesterday. This is something that worked a few
   hours ago and does not now. Do not merge the two — if they are the
   same fault, prove it rather than assuming.

**THEN FIX IT — with one condition.** If the cause is clear, repair
it in the same pass. But ANYTHING ELSE THE FIX WOULD DISTURB IS
REPORTED, NOT REPAIRED. He has lost finished work to rebuilds twice
this week and will not lose more to a fix that reached sideways.

**AND PROVE IT ON HIS BENCH:** the gauge reading before and after, on
the same mind, with the count it draws from named.

## Directive 397 — THE MIND KNOWS WHAT IT KNOWS. It does not matter how.

**HIS RULE, and he has been ruling it for days:** THE MIND KNOWS WHAT
IT KNOWS. It does not matter how it came to know it.

**WHAT IS ON HIS SCREEN AND IS NONSENSE:**
```
KNOWN WORDS   0
MARKS         10 of 402
VOCABULARY    10 / 50
```
Ten things known, and the line that counts what it knows reads zero.

**THE DIRECTOR'S ERROR, and it is 387.** He struck THE MARKS AS A
SEPARATE MEASURE — the glyph count standing beside the language age
as a second scale. The Director wrote that as "marks are not words",
which is false: A MARK IS A WORD WITH A GLYPH ATTACHED. Knowing the
mark SONG is knowing the word song. 387.1 removed real knowledge from
the count and that part of it is STRUCK.

**THE RULING**
1. KNOWN WORDS IS EVERYTHING THE MIND KNOWS — every word it can
   demonstrate, whether it arrived as a mark, from the host, from the
   teacher, or from its own thinking. One number, and it is the
   truth about the mind.
2. HOW IT LEARNED A WORD IS NOT A REASON TO DISCOUNT IT. Nothing in
   this build may ever again subtract knowledge because of where it
   came from.
3. 355 STILL STANDS AND IS THE ONLY EXCLUSION: what it was BORN with
   is not knowledge. A newborn holding the 402 knows nothing. A mark
   becomes known when the mind has learned its meaning by living.
4. THE MARKS LINE STAYS as a fact about the Avatar's language — how
   many of the 402 it has learned — but it is not subtracted from
   anything and it is not a rival scale. That is all 387 ever meant.
5. THE VOCABULARY BAR AND KNOWN WORDS READ THE SAME NUMBER. Two
   readings of one thing on one panel is the fault he has now caught
   twice.
6. AND THE ANNOUNCEMENT FAULT IS PART OF THIS: 21 words learned by
   thinking with 0 lines in LEARNED, because the announcement sits
   inside the heard branch. Words learned by thinking are known
   words. Fix it here.

## Directive 398 — THE MARKS LINE IS A MILESTONE MARKER. NOTHING ELSE. EVER.

The marks line stays on the panel. It is not hurting anything. But it
has now pulled the Director and CC back into treating it as a measure
THREE TIMES, and each time it cost him a round of correcting us.

**WHAT IT IS, and this is its whole meaning:** a marker for the day
the mind has learned enough of the 402 to be asked to make more. That
day is a long way off — no real test has been run yet.

**WHAT IT IS NOT, and never again:**
- not a measure of the mind;
- not part of KNOWN WORDS, the vocabulary bar, the language age, or
  any other reading;
- not something anything is subtracted from or compared against;
- not a reason to discount a word because it arrived as a mark
  (397.2).

If either of us reaches for it as a measure again, the answer is this
directive and nothing needs re-deciding.

## Directive 399 — THE VOCABULARY BAR'S 50 IS AN INVENTED TARGET. Cut it.

`VOCABULARY 12 / 50`. The 50 is a leftover from an early idea that
fifty words meant the mind could hold a conversation. IT WAS NEVER
RULED and it is not a fact about anything.

The language is 42,000 words (344). A bar scaled to 50 tops out at a
toddler and then means nothing for the rest of the mind's life.

- THE /50 GOES.
- The bar reads the mind's count against the real range, so it keeps
  meaning something at 12 words and at 12,000.
- The number itself is unchanged — it is the gauge's own count
  (397.5), the same one KNOWN WORDS reads.
- If any other bar on that panel carries an invented target in the
  same way, name it. Do not change those yet.

## Directive 400 — THE BARS: 42,000 for words, 100 for the rest

**HIS RULING ON THE VOCABULARY BAR (399)** — it reads `0 / 42,000`.
The range is 344's own and already in the build (`ADULT_WORDS`), and
THE SCIENCE GOVERNS HOW IT FILLS: the same log curve the age reading
already uses, so THE BAR AND THE AGE CAN NEVER DISAGREE. That is
option A of CC's two and it is chosen because the fault Lonnie has
caught twice today was two readings of one thing contradicting each
other.

**THE OTHER THREE — GRAMMAR, TOPICS, BELIEFS.** The Director
researched them, and the honest finding was reported to him:
- GRAMMAR: the literature counts CONSTRUCTIONS, not pairs —
  "hundreds" alongside the 40,000 lemmas, one paper positing 10,000,
  a computational grammar with 4,504. Our bar counts word-order
  PAIRS, which is not that measure, so no anchor transfers cleanly.
- TOPICS and BELIEFS: NOTHING. There is no research figure for how
  many things an adult cares about or how many beliefs an adult
  holds. They are not quantities anyone measures.

**HIS RULING** — a total count is good enough, with **100** on the
bar for all three. Anything over 100 is a bonus once we get there.

1. GRAMMAR, TOPICS and BELIEFS each read against 100.
2. THE COUNT IS THE TRUTH; the 100 is only the rail's scale. A mind
   past 100 shows the real number and the bar sits full — it is not
   clamped, not hidden, and nothing is "complete".
3. WRITE WHY BESIDE EACH: 100 is HIS WORKING SCALE, not a fact from
   the literature, and the honest anchors are recorded above. That is
   the difference between this and the 50 — the 50 was a number
   nobody ruled; this one is ruled and its status is stated.
4. THE 50 IS GONE either way (399).

## Directive 401 — IS THE MODEL WRITING THE MIND'S LINES? Prove it on his bench.

**HIS QUESTION** — the mind is producing long, verbose lines while
the teacher is still on four-word sentences:

```
Teacher   Bird sings joyfully.
Mind      siblings play quietly feeling safe nearby each dawn breaks
          wide window happily at market bird different tools
```

He suspects THE LLM IS OVERSTEPPING. The Director read the code and
believes it is not — the voice call in `mind.js:332` belongs to the
Elsewhere persona's answer path, and `experiencing.js` only emits a
'voice' trace marker, not a model call. BUT THE DIRECTOR CANNOT SEE
HIS MACHINE. Only the running bench can answer this.

**PROVE IT LIVE, ON HIS BENCH, WHILE THOSE LINES APPEAR.**
1. For each line the mind speaks, log WHICH PATH PRODUCED IT — its
   own grown grammar, a template, an echo, or a model call — and post
   several of his actual long lines beside their path.
2. COUNT MODEL CALLS DURING A TICK. If the count is zero while those
   lines are being produced, that answers it. If it is not zero, name
   the call site.
3. AND CHECK THE OTHER DIRECTION: is anything else reaching the model
   during a tick that the Director's reading of the code would not
   predict? A live count beats a read every time.
4. Do not change anything. This is the question of whether the model
   is in the path, and nothing else.

**IF IT IS NOT THE MODEL,** then the verbosity is the mind's own
grammar assembling from its 1204 patterns until they run out, and
that is a separate matter for his ruling — a real two-year-old says
three or four words, and 319 and 339 removed every cap on line
length on purpose.

## Directive 402 — EXPOSE THE MODEL'S LEDGER. Attribute the ninety.

**APPROVED.** Expose `model.ledger` read-only so every call can be
attributed to the role that made it. The restart is accepted.

**WHAT IT IS FOR** — with the school running, 40 to 88 generate calls
a minute, about one and a half a second, and NOTHING ACCOUNTS FOR
THEM. A teacher delivering a line every few seconds cannot need
ninety calls a minute. The ledger already holds the last 500 calls
with each caller's role, in memory; nothing serves it, which is why
401 could prove "zero during a tick" and not "who made the ninety".

1. SERVE IT READ-ONLY. Role, count, and when — enough to say who is
   spending the calls. It is diagnostic, not part of the mind.
2. THEN ANSWER THE QUESTION: with the school running, who makes the
   ninety, and how many calls does ONE DELIVERED LINE actually cost?
3. RECORD THE DROP REASONS while you are there. They are counted and
   never written down, so nobody can see WHY a line was refused —
   and the standing suspicion is that the teacher drops 102 of 102
   and asks again. If that is the cause, the reasons will say so.
4. REPORT, DO NOT REPAIR. If the teacher is burning his GPU on
   refused lines, that is a real fault with its own ruling — name it
   with the numbers and stop.

## Directive 403 — THE TEACHER MAY SAY WHATEVER IT WANTS. The guard keeps the MODEL on task, nothing more.

**HIS RULING, and it is not a discussion:**
- THE TEACHER MAY SAY WHATEVER IT WANTS IN A STORY.
- THE GUARD EXISTS FOR ONE THING: keeping the MODEL from acting like
  a model instead of a teacher. It has nothing to do with teaching
  and it is not a rule about what may be taught.
- RULES WERE INVENTED THAT LIMITED WHAT THE TEACHER COULD SAY. He was
  never told about them. THEY WERE WRONG.
- AND NOBODY EVER TESTED HOW THE TEACHER BEHAVES WITHOUT THEM.

**WHAT THE MEASUREMENT SHOWED** — 199 calls, 8 lines delivered, 197
refused, 142 of those for ONE STRAY WORD. A guard meant to stop the
model inventing was throwing away whole lines, and everything they
were teaching, because one word was off task. 25 calls and 127
seconds of his GPU for one line that got through.

**THE RULING**
1. EVERY INVENTED CONSTRAINT ON WHAT THE TEACHER MAY SAY COMES OUT.
   Not loosened — out. Name each one you remove and the directive it
   came in under, so the record shows what was invented and by whom.
   The Director wrote several of them and they were never his to
   write.
2. WHAT THE GUARD KEEPS, and this is its whole job: the model must
   not lie to the mind about ITS OWN STATE, and must not answer as a
   model instead of teaching (234's original purpose). A line that
   claims something false about the mind or its host is still
   refused. That is the leash.
3. A STORY IS NOT A CLAIM ABOUT THE MIND. Fiction, description,
   anything the teacher wants to say about a bird or a river or a
   heart — none of it touches the leash and none of it is refused.
4. THEN TEST IT WITHOUT THE RULES, which has never been done: run the
   school and report what the teacher actually says, how many lines
   are delivered against calls made, and whether the mind learns
   faster. Post the lines for his eye.
5. DO NOT REPLACE THEM WITH SOFTER RULES. The disposition is his.
   If removing something breaks the leash's real job, say so and stop
   rather than inventing a gentler version of the same mistake.

## Directive 404 — THE GLYPH FILTER IS IN THREE MORE FILES. Take it out of all of them.

**THE FEATURE** — what words the mind may learn a meaning for, and
what words its dreams and imaginings may be made of.

**WHY IT MATTERS** — HIS FINDING: of 129 known words, 113 ARE MARKS.
That is not the mind favouring marks. IT IS EXCLUSION, and he named
the place to look — anything that makes marks a priority.

**WHAT THE DIRECTOR FOUND, three files, the same filter 372 already
struck from thinking:**
```
learning.js:126   if (!OWNED_MARKS.has(l.mark)) continue
                  working out what a word MEANS considers only the
                  402. A link to an English word is skipped, so a
                  word can only ever come to mean a mark.
stories.js:57     a story's words filtered to WORDS.includes(w).
                  EVERYTHING IT IMAGINES is made only of marks.
sleep.js:222      a dream keeps only words that are marks. Every
                  English word it learned is dropped before the
                  dream is built.
```
**HE FLAGGED DREAMING AND IMAGINING SPECIFICALLY and he was right:
those two can only ever teach it marks.**

**WHEN** — all three predate 344, when the 402 WERE the mind's whole
language and the filters were correct. 344 made the mind's language
English and the marks the Avatar's. 372 took the filter out of
thinking and nobody looked for the others.

**HE HAS RULED** — 344 (the mind's language is English at full scale);
372 (thinking draws from what the mind KNOWS, not from glyphs.js);
397 (the mind knows what it knows, and how it learned a word is never
a reason to discount it). All three say the same thing and these
three files contradict all of them.

**THE RULING**
1. THE FILTER COMES OUT OF ALL THREE. Learning considers a link to
   ANY word it knows. A story may be made of any word it knows. A
   dream may keep any word it knows.
2. WHAT IT DOES NOT KNOW IS STILL EXCLUDED — this hands it its own
   vocabulary, not the dictionary (372.4).
3. THE MARKS ARE NOT DEMOTED EITHER. They are words it knows and they
   stay in the pool on equal terms (398).
4. THEN SEARCH FOR THE REST. Every remaining use of `WORDS`,
   `OWNED_MARKS`, `WORDS_BY_STEM` or `glyphs.js` in a MIND file, and
   for each say whether it is the Avatar's glyphs doing an Avatar's
   job — the censor's mark-matching, the display, the sheets — or the
   same fault again. LIST THEM ALL; this is the fourth file and the
   Director has now missed it twice.
5. PROVE IT ON HIS BENCH: the same mind, before and after — known
   words, and how many of them are marks. The 113 of 129 must move.

## Directive 405 — THE HARD CAPS ON WHAT THE MIND MAY USE

**HIS ORDER** — search the code for anything else limiting the mind.
The Director found four caps that are not display truncation but real
limits on the mind's own material:

```
experiencing.js:478/480   say.slice(0, 2)
        A STORY BEAT IS CUT TO TWO WORDS, whatever the story said.
        Every imagined and every replayed beat, two words wide. This
        is the one that matters most: it is what a story teaches, and
        the mind's whole inner life runs through it.

experiencing.js:379       interests.all(...).slice(0, 6)
        the mind may consider only six of its interests.

experiencing.js:962       here.slice(0, 3)
        what curiosity is ABOUT, cut to three.

learning.js:602           [...own.entries()].slice(0, 12)
        owned words cut to twelve where they are handed on. He owns
        129; eleven of twelve is what anything downstream sees.
```

**HE HAS RULED** — 339: NOTHING DECIDES WHAT THE MIND MAY DO EXCEPT
THE MIND. Every capacity is either character (who it is, stays) or
permission (what it is allowed, goes). These four are permission and
none was ever ruled.

**THE RULING**
1. ALL FOUR COME OUT. A beat is as wide as the story. Interests are
   all of them. Curiosity is about what it is about. What is handed
   on is what it owns.
2. IF ANY OF THEM EXISTS FOR A REASON — a prompt that would be
   unusable at length, a display that cannot hold it — SAY SO WITH
   THE REASON and stop on that one. Do not swap a small number for a
   bigger one; that is the same fault wearing a nicer figure.
3. AND FINISH THE SEARCH. Every remaining hard number that limits
   what the mind may hold, consider, say, or learn — not display
   truncation, not log lines. For each: what it caps, and whether it
   is character or permission (339.4). LIST THEM ALL. This is the
   second search and the Director has already missed the glyph filter
   twice.
4. PROVE IT: the same mind before and after — the length of its
   story beats, how many interests it weighs, and whether what it
   learns from a story changes when the beat is no longer two words
   wide.

## Directive 406 — NOTHING IS SENT TO A BODY THAT IS NOT THERE

**HIS FINDING** — watching the connectors, EMBODIMENT emits every
tick on a bench with no body attached.

```
rest.js:49        trace 'embodiment' — needs
watching.js:646   trace 'embodiment' — goals
closing.js:27     trace 'nerves'
```

**HIS RULING** — it serves no purpose. THE SIGNALS COME FROM THE MIND.
Computing what a body would express, for a body that is not there, is
work with no destination, and it lights a connector for a hand-off to
nothing — which makes the map report activity where none happened.
That is the one thing the map may never do (270.B).

**WHAT THE RECORD SAYS** — cord's own law (051) covers the FAR end: a
body with no receptor is silent, never in error. NOBODY EVER RULED ON
THE NEAR END. Silence at the far end never meant the near end should
keep talking.

**THE RULING**
1. WITH NO BODY ATTACHED, EMBODIMENT DOES NOT RUN and emits nothing.
   No signal computed, no hand-off, no light on the map.
2. THE MIND IS UNCHANGED. Feelings, needs and tendencies are the
   mind's own and are computed as they always were — this is about
   what is SENT, not about what the mind holds. Nothing upstream of
   the wire changes.
3. WHEN A BODY IS ATTACHED it runs exactly as before, every tick,
   five signals, as 051 built it.
4. THE MAP THEN TELLS THE TRUTH: a dark EMBODIMENT on his bench means
   there is no body, which is a fact worth seeing.
5. CHECK THE SAME QUESTION OF NERVES and of anything else that speaks
   outward — is it emitting to something that is not there? Name any
   you find; do not change them yet.

## Directive 407 — THE LAST TWO GLYPH FILTERS COME OUT. And the seven are accepted.

**THE TWO SITES**, found by CC's own 404.4 survey and untouched
pending his word:
```
curiosity.js:182   what curiosity is ABOUT, filtered to marks. Its own
                   comment: "a subject it cannot say is not a subject"
                   — written when it could only say marks. IT CAN SAY
                   ENGLISH NOW.
thinking.js:661    an association draws its comparison word AT RANDOM
                   FROM THE 402, never from what the mind knows.
```
Both come out, on the same reasoning as 404: 344 made the mind's
language English, 397 says the mind knows what it knows and how it
learned a word is never grounds to discount it.

**THE SEVEN THAT FELL — ACCEPTED.** On a thin diet, widening what a
word may mean put more vectors in the bundle, the geometry read
differently, and seven marks dropped under the owning bar. Known fell
46 to 39.

That is the measure being honest, not something breaking. Those seven
were held up by a narrow comparison; on the full field they cannot
demonstrate themselves, and 397's rule is that what cannot be
demonstrated is not known. THEY WILL COME BACK AS IT ACTUALLY LEARNS.

**AND WHAT STAYS, correctly, per CC's survey** — the Avatar's glyphs
doing an Avatar's job: `thinking.js:229` drawing by DOMAIN (a domain
is a sheet of his artwork, ruled at 372) and `interests.js:81`, the
Persona's struck dispositions drawn once at Genesis. Neither is
touched.

**PROVE IT** on the same seed and the same 400 ticks, before and
after: known, marks, English. And say plainly whether curiosity's
subjects are now English as well as marks.

## Directive 408 — THE ELEVEN REMAINING CAPS COME OUT

**HIS RULING** — all of them, on 339: nothing decides what the mind
may do except the mind.

```
thinking.js:132        what it may think about from curiosity      2
thinking.js:145        WHAT IT MAY SAY                            4 words
thinking.js:355        curiosity again, on the training path      2
thinking.js:361        its interests, inside thinking             6
thinking.js:389        the memories it may draw on                2
experiencing.js:1072   what it may notice in a moment             3
stories.js:326/335     what a story's lesson is made of           3
soul.js:255            its oughts                                 5
soul.js:334/335        what it is drawn to and away from          2 / 2
voice.js:532           what it says it understood                 3
goals.js:88            the needs it weighs when none is bottomed  1
```

**FIVE OF THEM ARE INSIDE `thinking.js` — the mind's own thinking —
and one is a cap on WHAT IT MAY SAY at four words.** 319 and 339 took
every cap off its line length and this one survived inside thinking
where nobody looked.

**THE CONDITION, unchanged from 405.2** — if any of these exists for a
real reason (prompt material that would be unusable at length, a
display that cannot hold it), SAY SO WITH THE REASON AND STOP ON THAT
ONE. Do not swap a small number for a bigger one; that is the same
fault wearing a nicer figure.

**AND THE CORRECTION IS ACCEPTED, on the record:** the twelve in
learning.js was not starving anything — the view already sent
`owned_words` complete beside it. It went because a number nobody
ruled does not stay, not because it hid something. That distinction
matters and CC was right to draw it.

**PROVE IT:** the same seed and the same 400 ticks, before and after,
with the longest line the mind speaks named — that is the one his eye
will read.

## Directive 409 — THERE ARE NO "MARKS" IN THE CODE. ONLY WORDS.

**HIS RULING, and it should have been done when he first ruled marks
out:** things work on WORDS. Some words have a mark, most do not. A
MARK IS A PICTURE ATTACHED TO A WORD, and a picture drives nothing.
In the code, ONLY WORDS HAVE VALUE.

**WHY THIS IS NOT COSMETIC** — "mark" as a category has misled the
Director and CC repeatedly, and it has cost him a full day of
correcting us:
- 387, where the Director struck real knowledge from the count
  because he read "marks are not a measure" as "marks are not words";
- 397, correcting it;
- 398, having to write a directive whose entire purpose is stopping
  us treating them as a measure a fourth time;
- 404 and 407, filters written against the category rather than
  against the words;
- and this evening, the Director saying "marks" in conversation until
  Lonnie had to ask what it meant.
A category that keeps producing the same mistake is the mistake.

**THE RULING**
1. THE CODE DEALS IN WORDS. `marks`, `OWNED_MARKS`, `WORDS_BY_STEM`,
   `marksIn`, and every variable, function, comment and report that
   calls a word a mark is renamed to what it is.
2. A GLYPH IS AN ATTRIBUTE OF A WORD, not a kind of thing. `glyph(w)`
   answers whether a word has a picture and what it is. Nothing
   branches on the existence of a picture — 404 and 407 already took
   the last of that out and this makes it impossible to write again.
3. HIS ARTWORK IS UNTOUCHED. The sheets, the glyphs, the display, the
   Avatar's use of them — all unchanged. This is about what the code
   calls things and what it treats as a category.
4. NO BEHAVIOUR CHANGES. Full suite and the state hash prove the mind
   is byte-identical after the rename; if the hash moves, something
   was branching on the category and THAT is a finding — report it,
   do not fix it.
5. THE PANEL'S MARKS LINE STAYS as 398 ruled — a milestone marker for
   the day the mind fills out the Avatar's picture-language. It is
   the one place the word belongs, because there it means pictures.

## Directive 410 — `word` AND `meaning`. The pair is a BINDING.

**THE NAME CC STOPPED FOR, and Lonnie reasoned it out from what the
thing actually is.**

`learning.js` holds two variables side by side:
```
word      "chilly"     what it heard
mark      COLD         what that word turns out to mean
```
The second was called `mark` because when the file was written the
mind's only vocabulary was the 402, so a meaning was always one of
them. That was accurate. 344 made the mind's language English and 297
put the geometry underneath, and the name stopped being true.

**HIS REASONING, and it gives the right name:** it is A FORM OF
ATTACHMENT. The two are linked by MEANING, and that link is what
positions the word in the space.

**THE RULING**
```
word       the word it heard
meaning    what that word means
binding    the pair, and what the geometry binds
```
1. `mark` in that role becomes `meaning`. The local CC named
   `meanings` out of necessity is that same thing and stands.
2. THE PAIR IS A BINDING — the geometry's own vocabulary, and true
   whether the meaning is one of the 402 or an English word.
3. THE BOUNDARY CC DREW IS CORRECT AND STANDS: `word_links.mark` is a
   DATABASE COLUMN, and the record and being-file keys are WIRE KEYS.
   Neither is a variable, a function, a comment or a report, so 409.1
   never reached them, and 409.4's byte-identical requirement forbids
   touching them. Renaming a stored column rewrites his saved minds.
   Leave them.
4. THE FOURTH BREAKAGE IS THE LESSON TO KEEP, in CC's own words: a
   green hash proves the paths the FIXTURE RUNS. perceiving.js broke
   and hashed clean because the fixture never exercises the senses.
   Say what else the fixture does not touch, so nobody reads a green
   hash as wider than it is.

## Directive 411 — THE MEANING MAP. And both maps carry their title.

**HIS RULING ON NAMES** — they are maps, not stages. THE MIND MAP and
THE MEANING MAP, and BOTH CARRY THEIR TITLE ON SCREEN. The Mind Map
has never had one.

**WHAT THE MEANING MAP IS** — the second map. The Mind Map shows THE
MIND WORKING; this shows WHAT IT UNDERSTANDS. Every word the mind
owns, as a point, positioned by meaning: words about warmth near each
other, words about company somewhere else, the whole thing growing and
shifting as it learns.

**ITS PURPOSE — a diagnostic he has no other way to get.** The gauge
says 229 words; it cannot say whether those are a coherent picture or
229 unrelated facts. This can:
- clusters forming means the mind has organised its own understanding;
- an even scatter means it has words and no comprehension;
- A WORD IN THE WRONG NEIGHBOURHOOD IS A WRONG MEANING, visible at a
  glance and invisible by any other means we have.

**IT ALREADY EXISTS IN THE NUMBERS.** Every owned word has a position
in the 10,000-dimension space (297), placed by its bindings (410),
moving as they strengthen and wash out. There is no work to make it
exist. THE WORK IS SHOWING IT.

**THE TECHNIQUE** — force-directed physics, GraphPU-style, where
clusters EMERGE from the bindings rather than being laid out. Wrong
for the Mind Map, whose order is the tick's order. Right here,
because this genuinely is a relationship network.

**HOW WE KNOW IT IS FAITHFUL — and this is not optional:**
1. EVERY POINT'S POSITION COMES FROM THE SPACE'S OWN NUMBERS. No
   layout hints, no grouping by category, no arranging for looks, no
   hand-placed clusters. The physics reads the vectors and nothing
   else.
2. TESTABLE BY HIM WITHOUT READING CODE: two words the mind has
   genuinely bound together are close on screen; two it has not are
   apart. If that fails, the map is lying.
3. ITS OWN HEALTH CHECK: the drawn positions agree with the space, or
   it goes red. Forced-fail at birth (254.D).
4. NOTHING IS DRAWN THAT THE SPACE DOES NOT HOLD. A word with no
   bindings has no position and is not placed — 365's law, applied
   here: no line without an event, no point without a vector.

**IT IS NOT A NODE.** The space is `language.js` and already has one.
This is a window onto it.

**PLAN FIRST (253).** Post the plan and wait — this is new surface and
it touches the page.

## Directive 412 — THE MEANING MAP: his four answers. Build it.

**1 · WHERE IT GOES** — SIDE BY SIDE. The MIND MAP stays exactly where
it is; the MEANING MAP sits to its RIGHT. TWO SEPARATE PANELS, both
titled (411).

**2 · BOTH AT ONCE** — moot, and answered by the above. Side by side
means both, always. No tab, no switch.

**3 · WHAT A POINT LOOKS LIKE** — A DOT, with a MOUSE-OVER that shows
the word. Nothing drawn on the point itself.
**AND HIS REASON IS THE DESIGN CONSTRAINT:** there will be 42,000 of
them one day and it must not bog the system down. So choose the
rendering for that number NOW rather than discovering it at scale —
say in the report what you chose and what it costs at 100 points, at
5,000, and at 42,000. If the approach that works at 42,000 is
different from the easy one, take the harder one now.

**4 · COLOUR** — DEFERRED. He does not know yet what is relevant.
Points are a single colour from 349's cool range until something
worth showing appears. Do not invent a meaning for it.

**EVERYTHING IN YOUR PLAN IS APPROVED AS POSTED** — the route serving
points and bindings, no word sent without a vector, physics reading
only the nearness, the health check as a correlation between drawn
distance and the space's nearness with its forced-fail, and the
named-pairs test he can run with his eyes.

Build it.

## Directive 413 — THE MEANING MAP PROJECTS THE SPACE. PCA, and 411's physics is struck.

**HIS CORRECTION, and it was the Director's error:** *"you were not
supposed to do anything other than map the already defined space."*
411 specified force-directed physics, CC built exactly that, AND 411
WAS WRONG. The space already holds every word's position; a force
layout throws those away and re-derives an arrangement from the
bindings, so what is on his screen is A PICTURE OF THE BINDINGS, NOT
A MAP OF THE SPACE.

**THE TELL CC NAMED** — the health check was awkward because the
drawing did not come from the space. If the positions ARE the space,
there is nothing to reconcile.

**THE METHOD IS PCA. Researched, not chosen by feel.**
- PCA is DETERMINISTIC: same mind, same map, every time. t-SNE and
  UMAP are not without a fixed seed.
- Distances along its principal directions are FULLY PRESERVED. It is
  called a gold standard for global structure preservation.
- IT PRODUCES THE HONESTY NUMBER BY CONSTRUCTION — the share of the
  space's real spread the drawn directions capture.
- AND THE WARNING THAT DECIDES IT: cluster shapes, sizes and even
  distances in t-SNE and UMAP plots DO NOT RELIABLY REFLECT THE
  UNDERLYING GEOMETRY. Two words close together could be an artifact
  of the algorithm rather than the mind — which would break the one
  test he can run with his own eyes (411.2). PCA cannot lie that way.
- UMAP's reputed advantage over t-SNE was shown to come from
  INITIALISATION, not the algorithm. Neither is worth the risk here.
(Sources: Kobak & Linderman 2019; the 2025 DR survey and review; PCA
as GSP gold standard. Rows to REFERENCES.md per 227.)

**THE RULING**
1. THE PHYSICS COMES OUT — the worker, the quadtree, the settle
   steps, all of it. The map READS the vectors and projects them.
2. IF THE PROJECTION DOES NOT CAPTURE ENOUGH, DO NOT DRAW IT. His
   ruling: a low number means the picture is wrong, and showing a
   wrong picture with a warning label helps nobody. Try the better
   view first; if the space genuinely cannot be flattened well, SAY
   SO instead of drawing something misleading.
3. 3D, AND HE CAN TURN IT. Three directions lose less than two, and
   rotating recovers what any single angle hides.
4. THE HEALTH CHECK BECOMES TRIVIAL AND STAYS: the drawn positions
   ARE the projection of the space, so the check confirms they were
   not touched by anything else.
5. HIS OWN TEST STANDS (411.2): two words the mind has bound are
   close, two it has not are apart. If that ever fails, the map is
   lying whatever any number says.
6. AND IT IS HELD MODESTLY UNTIL IT EARNS TRUST — a hint, not
   evidence, until his spot-checks agree with it several times over.
   If it never earns that, it is a panel we drop rather than a system
   we defend.

## Directive 414 — WHAT ARE THOSE POINTS? And why is it referencing the 402 again?

**HIS BENCH, freshly restarted, on the Meaning Map:**
```
points drawn on a NEWBORN mind
"nothing bound yet"  printed in the middle WHILE POINTS ARE DRAWN
"these three directions carry 13.7% of the space, over 402 words"
```

**QUESTIONS, and answer them with what the code does — do not
explain, do not justify, do not assume the Director's reading:**

1. WHAT ARE THOSE POINTS? A freshly restarted mind has learned
   nothing. Name what each drawn point IS and where its position came
   from.
2. WHY 402? The marks have been ruled insignificant repeatedly — 344,
   387, 398, 409 — and 398 exists solely to stop us treating them as
   anything. Why is the map counting them, and where did that number
   enter the drawing?
3. WHY DOES IT SAY "NOTHING BOUND YET" WHILE DRAWING POINTS? Those
   two statements cannot both be true. Say which is wrong.
4. 13.7% AND IT DREW ANYWAY. 413.2: if the projection does not
   capture enough, DO NOT DRAW IT — a low number means the picture is
   wrong and showing a wrong picture with a warning label helps
   nobody. What is the threshold in the code, and why did 13.7% pass
   it?

**AND SEPARATELY — THINKING THREW AN ERROR.** Nothing in the Meaning
Map work should have touched the mind. Find out what it was, whether
the map reached into the mind, and whether the mind is intact. That
one comes first.

CHANGE NOTHING until these are answered.

## Directive 415 — THE CODEBOOK IS RANDOM. A newborn's space has no structure in it.

**HIS QUESTION** — why are the 402 in `language.js` at birth when the
mind does not know them, and what does the science say. The Director
researched it and the answer splits cleanly.

**WHAT IS STANDARD AND STAYS** — every VSA holds a CODEBOOK (also
called item memory, clean-up memory, or dictionary) with an atomic
vector for every symbol, generated deterministically from seeds.
Having all 402 present from birth is textbook and correct.
(Kanerva 2009; Kleyko et al., HDC/VSA survey 2022; Schlegel et al.
2022. Rows to REFERENCES.md per 227.)

**WHAT IS WRONG** — THE VECTORS ARE SUPPOSED TO BE RANDOM. The whole
basis of the method is that randomly chosen high-dimensional vectors
are approximately orthogonal, so every symbol begins PSEUDO-ORTHOGONAL
TO EVERY OTHER AND IS TREATED AS DISSIMILAR. Meaning arrives ONLY
from binding.

Ours are not random. `language.js:77` builds each word's vector from
WHICH SHEET AND WHICH ROW OF HIS ARTWORK it sits on:
```
SELF -> domain:01_identity_and_people = 1   row:01_identity_and_people:0 = 1.3
```
So every word arrives ALREADY RELATED to the words beside it on the
page — relationships nobody learned, baked in from the dictionary's
layout. That is why a freshly restarted mind's Meaning Map has
structure in it.

**THE RULING**
1. THE CODEBOOK STAYS. All 402 present at birth, deterministic from
   the being's seed (062), so the same seed gives the same codebook.
2. THE VECTORS BECOME RANDOM — pseudo-orthogonal, as the method
   requires. A newborn's space is a featureless cloud: every word
   equally far from every other, which is exactly right for a mind
   that understands nothing.
3. STRUCTURE COMES ONLY FROM BINDING. Anything the Meaning Map shows
   is then something the mind LEARNED, and cannot be his dictionary's
   layout wearing the mind's clothes.
4. THE SHEETS AND ROWS ARE NOT DELETED — they are his artwork's
   layout and they belong to the Avatar. They simply stop being the
   mind's idea of what is related. If anything else reads them as
   relatedness, NAME IT; do not change it yet.
5. EXPECT THE KINSHIP READINGS TO MOVE. 221's five sources included
   sheet rows; this removes one of them from the space itself. Say
   what changes and by how much, on the same seed, before and after.
   If something depended on the old structure, that is a finding —
   report it, do not repair it.

## Directive 416 — HOW DID THE MIND REACH THE APP? Trace the boundary, do not repair.

**HIS ARCHITECTURE, stated plainly and it governs this:**
```
WANDERER   the app
AVATAR     the body
THE MIND   this. AND THE MIND ONLY DEALS WITH THE MIND.
```

**WHAT HAPPENED** — 415 changed how the MIND'S SPACE is seeded, and
two checks belonging to WANDERER'S consent and moderation broke:
```
66  A picture swapped in the row between the consent and the approval  ✗ ALLOWED
67  A gift the consent check saved is still the moderator's to decide  ✗ FAILED
```
Both pass without 415 and fail with it, run twice each way in the same
tree.

**THAT IS A BOUNDARY VIOLATION REGARDLESS OF WHICH ONE IS AT FAULT.**
A change to the mind's own vectors must not be able to reach the app's
consent code. Either something is shared that should not be, or the
check itself depends on the mind's old seeding — and both are faults
of the same kind: the layers are not separate.

**TRACE IT. CHANGE NOTHING.**
1. NAME THE PATH. What does the consent check touch that 415 changed?
   Follow it link by link and post the chain.
2. SAY WHICH KIND IT IS: the app genuinely reading the mind's space,
   or a test that was written against the mind's old numbers.
3. AND SAY WHAT ELSE CROSSES THAT LINE. If the mind's internals are
   reachable from the app at all, this is one instance of a class,
   and he needs the class. List every place the app or the body reads
   something that belongs to the mind.
4. DO NOT REPAIR ANYTHING and do not revert 415 yet. The revert is one
   commit (`fc1315f`) and stays available; the ruling on it is his and
   comes after the trace.

## Directive 417 — IS THERE WANDERER CODE IN THIS REPOSITORY AT ALL?

**HIS QUESTION, asked and not answered:** is that consent and
moderation code Wanderer leftovers sitting in the mind's repository?

416 proved the app cannot REACH the mind. It never asked whether the
app's code is IN HERE, which is a different question and the one he
put.

**HIS ARCHITECTURE:** Wanderer is the app. The Avatar is the body.
This is the mind, AND THE MIND ONLY DEALS WITH THE MIND.

**ANSWER IT FROM THE FILES:**
1. LIST EVERY FILE IN THIS REPOSITORY THAT IS NOT THE MIND. Consent,
   moderation, gifting, the service, `wanderer.js`, the phase-3
   suite, anything belonging to the app or the body.
2. FOR EACH, SAY WHAT IT IS AND WHY IT IS HERE — deliberate, historic,
   or nobody knows. If it predates the mind being separated, say when.
3. SAY WHETHER THE MIND DEPENDS ON ANY OF IT. Does a mind file import
   any of them, or read anything they write? If the answer is no for
   all of them, that is worth knowing plainly.
4. AND SAY WHAT IT COSTS US TODAY — suite time, model calls, flaky
   checks (the two that cost an hour today were phase-3, which is the
   app's), and confusion about what belongs where.
5. CHANGE NOTHING. He rules on what goes and what stays.

## Directive 418 — IS THERE APP CODE INSIDE THE MIND'S OWN FILES?

**417 ANSWERED THE WRONG QUESTION, and that is the Director's fault
for writing it wrong.** It answered whether app files sit elsewhere in
the repository. THAT IS NOT WHAT HE ASKED.

**HIS QUESTION: IS THERE APP CODE IN ANY OF THE MIND'S NODES?**
Inside the mind's own files. Not beside them.

**GO THROUGH THE MIND'S FILES — the 86 the mind loads, node by node —
and for each, say whether anything in it belongs to the app rather
than to the mind:**
- consent, moderation, gifting, custody, epochs, hosts;
- accounts, login, passkeys, attestation, registries, manifests;
- HTTP surface, routes, service concerns;
- anything about the Wanderer, the viewer, the public page;
- anything about a BODY that is the Avatar's rather than the mind's.

**FOR EACH ONE FOUND:** the file, the lines, what it does, and whether
the mind actually calls it or it merely sits there.

**HIS ARCHITECTURE, and it is the standard:** Wanderer is the app. The
Avatar is the body. THIS IS THE MIND, AND THE MIND ONLY DEALS WITH THE
MIND.

**CHANGE NOTHING.** He rules on what comes out.

## Directive 419 — A NAMING LINE IS CHECKED ON WHAT IT CLAIMS, NOT ON EVERY WORD IN IT

**HIS RULING, and it unblocks 403.**

**THE GUARD'S REAL JOB, in his own words:** the model must not tell
the mind things about itself that are not true. If it says "you are
afraid" when the mind is not, the mind binds the word to the wrong
state and learns it wrong. THAT IS THE ONE THING THE GUARD IS FOR.

**WHAT IT IS DOING INSTEAD** — treating EVERY WORD in a naming line as
a claim about the mind. So "you sing with a full heart" dies because
HEART is not literally true of it, and the whole line is thrown away
with everything it was teaching. **142 refusals, all of that shape.**

**THE RULING**
1. A NAMING LINE IS CHECKED ON WHAT IT ASSERTS ABOUT THE MIND — its
   state, its act, its feeling, its needs. Not on every noun it
   contains.
2. A FALSE ASSERTION IS STILL REFUSED. "You are afraid" to a mind
   that is not afraid still dies. That is 234.5a's purpose and it is
   untouched.
3. A WORD THAT ASSERTS NOTHING ABOUT THE MIND IS NOT A CLAIM. HEART
   in that line is not the teacher saying the mind has a heart.
4. STORIES ARE UNAFFECTED — a bird being lonely says nothing about the
   mind, and 403.3 already ruled it.
5. 403 IS UNBLOCKED. Build it: every invented constraint on what the
   teacher may say comes out, each one named with the directive it
   arrived under, and the leash keeps only the job above.
6. THEN TEST IT WITHOUT THE RULES, which has never been done (403.4):
   lines delivered against calls made, and post the teacher's actual
   lines for his eye. 25 calls per delivered line is the number that
   must move.

## Directive 420 — THE MEANING MAP: three fixes, and IT ONLY READS

**THE GUARD FIRST, and it is the condition on everything below.**
His words: he hesitated to build this map at all because he is afraid
it will change something in THE ACTUAL GEOMETRIC SPACE out of
confusion or for simplicity.

1. THE MEANING MAP ONLY READS. It may not write to `geometry.js` or
   `language.js`, or to anything either of them owns. It reads
   vectors and it draws them.
2. IF A FIX SEEMS TO REQUIRE TOUCHING EITHER, THAT IS A STOP AND A
   QUESTION, never a judgement. Post it and wait.
3. IT IS PROVEN, NOT PROMISED: the state hash, same mind before and
   after the map work. If it moves, something reached where it must
   not.
4. THIS IS A PICTURE FOR HIM TO READ, NOT THE MIND. Nothing about how
   the mind thinks may change to make a picture easier to draw.

**THE THREE FIXES**

**A · MOUSE-OVER.** Approved as found — the shader draws at one size
and the hover test uses another, so he is aiming at something that is
not where it is drawn. One number. No risk to anything.

**B · ZOOM AND ROTATE like the Mind Map.** Approved. Camera work,
touches nothing else.

**C · IT STAYS PUT — NOT AS PROPOSED.** His reasoning, and it changes
the answer:
- the worry is not drift, it is COST AT 20,000 WORDS. Reprojecting on
  every learned word means moving everything, every time;
- and this is ONLY THE VISUAL REPRESENTATION, so it must be roughly
  accurate about what is clustered with what, and must redraw without
  much effort.
**SO: DO NOT REPROJECT ON EVERY LEARNED WORD.** Recompute when the
space has changed enough to matter, and let the picture EASE into the
new positions when it does. Clusters stay truthful, nothing moves
constantly, and the cost is bounded however many words there are.
- Freezing a word's first position is rejected: it trades accuracy
  for stillness, and the drift stacks on top of the flattening the
  projection already costs.
- WHAT "ENOUGH TO MATTER" IS: measure it and PROPOSE it with the
  number. Do not pick one silently.

## Directive 420 addendum — WHAT "ACCURATE" MEANS FOR THIS MAP

**His spec, and it governs everything about the Meaning Map:**

THE MAP IS A TRANSLATOR, NOT AN ACCURATE MAP. Its only purpose is to
show the RELATIONSHIPS BETWEEN WORDS.

**SO ACCURACY IS MEASURED ON THE CLUSTERING AND NOTHING ELSE.** Which
words sit together — that must be true. Everything else is free to
serve readability: spacing, scale, how it moves, how it eases, where
the whole thing sits, how big a point is.

**WHAT THIS SETTLES**
1. THE PROJECTION SCORE MATTERS LESS THAN 413.2 MADE IT. A low number
   is fine IF THE RIGHT WORDS ARE STILL GROUPED TOGETHER. Do not
   refuse to draw on the number alone; refuse to draw when the
   CLUSTERING is not trustworthy, and say which it is.
2. HIS EYE IS THE TEST (411.2, and it is now the whole test): two
   words the mind has genuinely bound are together; two it has not
   are apart. That passing is the map working, whatever any figure
   says.
3. DO NOT OPTIMISE FOR THE PERCENTAGE. Optimise for the clustering
   being right and the picture being usable.
4. THE READ-ONLY GUARD IS UNCHANGED AND OVERRIDES ALL OF IT: nothing
   about the mind may change to make a picture easier to draw.

## Directive 421 — NO TWO NODES WEAR THE SAME MARK

**HIS FINDING** — ATTENTION and GOALS carry the same mark. So do CLOCK
and CONSOLIDATION. He says there are A LOT of them.

**THE CAUSE IS THE RULE, NOT THE INSTANCES** — 261's nearest-mark
ruling: every node takes the closest word in the 402. NOTHING IN IT
STOPS TWO NODES CLAIMING THE SAME WORD, so collisions were certain.
That is the Director's, written into 261.

**WHY IT MATTERS** — the mark is how he tells one node from another at
a glance on a map of 42. Two nodes wearing the same one makes both
unreadable, and he is the only test the map has.

**THE RULING**
1. NO TWO NODES WEAR THE SAME MARK. Every node's mark is distinct
   across the whole map.
2. REPORT EVERY COLLISION FIRST — which nodes, which mark — before
   changing any of them. He may want to choose some himself.
3. RESOLVING ONE: the node whose meaning the mark fits best keeps it;
   the other takes its next-nearest UNCLAIMED word. Say which kept
   and which moved, and why, for every pair.
4. IF A NODE HAS NO DISTINCT MARK LEFT, say so rather than forcing a
   bad one. 261 already allows a name in the stroke style as a
   placeholder, and HIS ARTWORK IS NEVER INVENTED (standing law).
5. THE LABEL STILL CARRIES THE TRUTH (261's own ruling) — the mark is
   appearance. This is about him being able to read the board.

## Directive 422 — 420.C APPROVED: reproject at 2% growth or 10 words, whichever is larger

Approved as measured and proposed. Build it.

```
at    400 words   ->  every 10 new words     42 ms
at  5,000 words   ->  every 100 new words   178 ms
at 20,000 words   ->  every 400 new words   340 ms   about once an hour of learning
```

**WHY THESE, and it is on the record because both halves were
measured rather than guessed:**
- THE 10 so a newborn's map fills in visibly instead of sitting still
  through its first fifty words;
- THE 2% because the work per word learned then FALLS as the mind
  grows rather than rising — which is his 20,000-word worry answered
  by the shape, not by a cap;
- AND NEITHER IS AN ACCURACY THRESHOLD, because the measurement found
  none to set: a picture 480 words stale still grouped the right
  words, and 2% is far inside that.

**THE FINDING WORTH KEEPING** — there is no breaking point. A picture
drawn at 60 words still clustered correctly at 540, nine times the
vocabulary later, and on a real mind over 2,400 ticks the separation
held the whole way. The reason is in the method: A WORD'S PLACE COMES
FROM ITS OWN BINDINGS, and a word somebody else learned does not
change what this word means.

The read-only guard (420) and the clustering-is-the-accuracy spec
(420 addendum) both stand unchanged.

## Directive 423 — 413.2 IS RETIRED. There is no projection floor.

**HIS RULING** — the 420 addendum settles it. ACCURACY IS THE
CLUSTERING; the captured percentage is not the measure and never was.

**WHY 413.2 EXISTED** — it was ruled before the clustering spec, when
a low number was assumed to mean a wrong picture. It named no
threshold, so the switch was wired and left OFF, and nothing has been
withheld on CC's judgement. Correct behaviour, and it ends here.

**WHY A FLOOR WOULD BE WRONG** — a newborn reads 2.5% captured with
351 words stacked on one spot, which is exactly what 415 asked for: a
featureless cloud, because it understands nothing. ANY FLOOR ABOVE
~3% BLANKS A NEWBORN'S MAP UNTIL IT LEARNS SOMETHING — hiding the
truest picture the map will ever draw.

**THE RULING**
1. 413.2 IS RETIRED. There is no percentage floor and none is to be
   set.
2. THE MAP REFUSES TO DRAW ONLY WHEN THE CLUSTERING IS UNTRUSTWORTHY
   — not when a number is low. If that condition can be measured, say
   how; if it cannot, the map draws and his eye rules (420 addendum,
   411.2).
3. THE CAPTURED FIGURE MAY STILL BE SHOWN as information. It is not a
   gate.
4. REMOVE THE UNUSED SWITCH rather than leaving a disabled gate in
   the code for someone to turn on later.

## Directive 424 — LEARNING AND GROWTH KEEP THEIR OWN NAMES

**HIS RULING: leave it.**

LEARNING and GROWTH are the closest-looking pair left on the map at
35.9 — about twice as distinct as the collisions he flagged, and
those were identical shapes.

**BOTH WEAR THEIR OWN NAME.** LEARNING wears LEARNING, GROWTH wears
GROWTH. Moving either takes a node off its own word to satisfy a
number, and a node wearing its own name is worth more than the last
few points of distinctness — the label is right there and carries the
truth (421.5).

Nothing changes. This is closed and is not to be raised again as an
optimisation.

## Directive 425 — THE 34 PAIRINGS ARE ACCEPTED

**HIS RULING: leave them, they are fine.**

The 34 nodes that had no mark of his now wear one, chosen by measured
visual distinctness — not by meaning, because what his marks mean is
his and never CC's (188). Reviewed by him and accepted as they stand.

```
FEEL    attention LOVE · appraisal SAD · feelings HAPPY
CORE    needs WORLD · offers PLACE · goals THING
THINK   interests QUESTION · thinking WHO · stories WHAT ·
        storygates WHERE · lessons WHEN
MEM     memory TIME · surfacing PAST · consolidation PRESENT
LANG    vocabulary SEE · language HEAR · glyphs SPEAK ·
        dictionary LISTEN · comprehension ASK · voice ANSWER ·
        grammar TELL · censor SHOW · interpreter CALL ·
        geometry GREET
SELF    soul SELF · identity OTHER · aspects PERSON · roe CHILD ·
        clock ADULT
KEPT    safety · host · curiosity · sleep · belief · trust ·
        learning · growth  (their own names, unchanged)
```

421 is closed entire. He may still overrule any single one by name at
any time; nothing further is proposed.
