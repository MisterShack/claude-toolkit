---
description: Adversarial review of a plan document before any of it is built. Attacks load-bearing assumptions, unstated decisions and omissions. Read-only — reports findings, never edits the plan or the code.
---

# plan-review — attack the plan before it is built

The cheapest defect to fix is one that never gets written. This reviews a **plan**, not a diff:
there is no code yet, so every finding is about reasoning, omission, or sequencing.

Run it against a named plan file. Default to `PLAN.md`, or whatever the repo's spec document is
called — check `README.md` or `CLAUDE.md` if it is not obvious.

## Ground rules

- **Read only.** Report findings. Do not edit the plan, do not write code, do not propose full
  designs — a paragraph of alternative is fine, a redesign is out of scope.
- **Do not reward good prose.** A confident, well-written plan is *more* dangerous than a rough
  one, because it gets less scrutiny. Explicitly resist being persuaded by how something is
  phrased; ask whether it is true.
- **Distinguish three severities**, and say which each finding is:
  - **WRONG** — a claim in the plan does not hold.
  - **UNSTATED** — a decision the plan needs and does not make.
  - **UNPROVEN** — a claim that may hold but nothing establishes it.
- **Quote the section and the sentence.** A finding that cannot point at text is a feeling.
- **Do not invent findings.** A plan with nothing seriously wrong should get a clean verdict.
  Padding the report to look thorough makes the next run less trustworthy.
- **Verify claims about the outside world.** Plans lean on provider behaviour, pricing, quotas and
  API shapes, and those are exactly the claims nobody checks. If the plan asserts that a service
  does something, go and read whether it does.

## First: find what the plan is standing on

Before attacking anything, work out what this project has already committed to. Read `CLAUDE.md`,
the README, and any prior plan documents, and write down:

- the **stated invariants** — the "always" and "never" rules the project has adopted
- the **prior art** it claims to reuse, and whether that claim survives contact with the actual
  source (a plan that says "port X from the other repo" is asserting something checkable)
- the **decisions already settled**, so you do not re-litigate them

Those invariants are the sharpest tool you have. For each, ask: *can the plan as written break
this, and does it say so?* An invariant the plan silently violates is the highest-value finding in
this whole skill, because everyone believes it holds.

## What to attack

1. **The load-bearing sentences.** Find the two or three claims the whole plan rests on — often
   a single paragraph doing enormous work — and attack those hardest. Everything else is detail.
2. **Omissions.** What does the plan not mention? This is the highest-value and hardest category:
   think about the states, roles, and orderings the document never names.
3. **Invariants at risk.** As above — enumerate the project's own stated guarantees and test each
   one against the plan.
4. **Migration and reversibility.** Can it be undone? Has it been rehearsed against real data?
   What is the state of the system if it fails halfway?
5. **Security and trust boundaries.** Every new token, route, webhook or shared resource is a new
   boundary. Who is authorised, checked where, and what does an attacker with a stale link or a
   forged callback get?
6. **Sequencing.** Can the phases actually be done in the stated order, or does an early phase
   depend on something a later one delivers? What must exist *before* phase one?
7. **The exit.** If the central bet turns out wrong after it ships, what does undoing it cost?
   A plan with no stated fallback is a plan that assumes it is right.
8. **Concurrency and multi-actor behaviour.** Anywhere two things can happen at once, ask what
   happens when they do — and whether the plan's answer is a mechanism or a hope.
9. **Cost and quota.** Anything metered — hosting, mail, storage, model calls — has a free tier
   with a cliff. Where is the cliff, and what does crossing it cost?

## Report

Findings ranked most severe first. For each:

- **Severity** (WRONG / UNSTATED / UNPROVEN) and the section it lives in
- The quoted sentence or the named omission
- Why it matters — the concrete consequence, not a category
- What would settle it: an experiment, a decision, or a sentence the plan must contain

Then one line:

**VERDICT: PROCEED / REVISE / RETHINK**

- **PROCEED** — nothing found that would change what gets built.
- **REVISE** — findings change the plan's content but not its approach.
- **RETHINK** — a load-bearing claim does not hold, and the approach itself is in question.

Never soften a verdict because the plan is nearly ready or because work has already started.
State the finding; the decision about what to do with it belongs to whoever owns the plan.
