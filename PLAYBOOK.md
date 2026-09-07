# THE PLAYBOOK — how we build, from here on

**Source: Anthropic's AI-Native SDLC Playbook, 21 August 2026.**
https://claude.com/blog/the-ai-native-sdlc-playbook

**His ruling: this is our bible for building apps.**

We arrived at most of it by being burned. This is the version written
down by people who did it on purpose, and where we differ from it we
should have a reason.

---

## THE ONE IDEA UNDER ALL OF IT

**Code is no longer the bottleneck. The steps around it are.**

Build collapses to hours. Plan, review and deploy still run at human
speed, so they become the constraint. **That is exactly this project:
the mind gets built in minutes and a day goes on everything else.**

---

## THE SIX STAGES, AND WHERE WE STAND

| Stage | What it says | Where we are |
|---|---|---|
| **PLAN** | Intent captured once, in the originator's own words, as `intent.md` | **WE HAVE THIS.** `INTENT.md`, in his words, version-controlled |
| **DESIGN** | Requirements and design in one session, guided by skills | **PARTLY.** Directives do this; we have no skills |
| **BUILD** | Nothing implemented without an accepted plan. `CLAUDE.md` holds what a new joiner needs | **YES** — 253 is plan-before-build, and CLAUDE.md exists |
| **TEST** | The agent verifies its own work before a human sees it | **YES** — the suite, the health checks, the state hash |
| **DEPLOY** | Layers of agentic review; hooks as approval gates | **YES** — the reviewer, the Critique, six hooks |
| **MAINTAIN** | Agents watch and write findings back as new intent | **NO.** Nothing watches |

---

## THE LINES THAT NAME OUR OWN FAULTS

**On CLAUDE.md** — *"If your CLAUDE.md is too long, Claude ignores
half of it because important rules get lost in the noise. Ruthlessly
prune. If Claude already does something correctly without the
instruction, delete it or CONVERT IT TO A HOOK."* And: **keep it under
a page.** Ours is far longer.

**On hooks versus rules** — *"Unlike CLAUDE.md instructions which are
advisory, HOOKS ARE DETERMINISTIC and guarantee the action happens."*
Every law we wrote by hand was broken at least once.

**On skills** — *"A skill is a control, though an advisory one... A
policy that must always hold needs something deterministic behind the
skill."* **THE SKILL MAKES VIOLATIONS RARE AND THE HOOK MAKES THEM
CLOSE TO IMPOSSIBLE.**

**On review** — *"the agent that wrote the code has no way to approve
it."* Separation of duties, which is why the Critique and the reviewer
are separate from CC.

**On repeated mistakes** — *"When Claude makes a mistake twice, the
correction goes into CLAUDE.md."*

**On the artifact chain** — *"Every stage commits an artifact the next
stage can read... The chain of commits is also the audit trail: who
asked for what, what the agent produced, and who approved it."*
**That is the relay, and we built it before reading this.**

---

## WHAT WE ARE MISSING, IN ORDER OF WHAT IT WOULD SAVE HIM

**1 · SKILLS.** Institutional knowledge as files the agent loads
automatically, at `.claude/skills/<name>/SKILL.md`. **Our design laws,
the presentation structure, the science-first rule — every one of
these is a skill we are re-explaining by hand.** The playbook's rule:
write a skill for knowledge that must be applied consistently.

**2 · `REVIEW.md`.** The review policy as a file at the repo root, in
passes, defining what counts as Important versus a nit. **We have been
ruling this by directive one clause at a time (476, 480, 482).** It
belongs in one file the reviewer reads.

**3 · EVALS.** A suite of 20-50 real tasks that runs whenever the
agent's configuration changes. **We change hooks, agents and
CLAUDE.md constantly and nothing regression-tests them.** The playbook
is blunt: that configuration steers the agent and deserves the
regression testing that code gets.

**4 · MAINTAIN.** Nothing watches the mind and writes findings back.
**His whole complaint — that he finds the faults himself — is this
stage missing.**

**5 · CAP THE NITS.** *"Report at most five nits per review;
summarize the rest as a count."* Eight findings a pass is what 476
was fighting.

---

## THE ONE PLACE WE DIVERGE ON PURPOSE

The playbook's chain is `intent.md` -> `spec.md` -> `plan.md` -> diff.
**Ours is one directive that carries all three**, because there is one
product owner, one director and one builder, and a spec handed between
three roles is the overhead this structure exists to avoid.

**That is a real difference and it should stay a decision rather than
become a discovery.**
