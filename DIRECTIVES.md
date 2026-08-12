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
