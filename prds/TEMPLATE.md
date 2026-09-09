# PRD: [Product / feature name]

**Owner:** · **Status:** Draft | In review | Approved | Shipped | Killed
**Last updated:** YYYY-MM-DD · **Engineering partner:** · **Design partner:**

> Every load-bearing claim is labeled **[measured]**, **[reported]**, **[inferred]**, or
> **[unknown]**. Unlabeled numbers are not evidence.

---

## 1. Summary

Three sentences. What we are building, for whom, and why now. If this section needs a paragraph,
the thinking is not done.

## 2. Problem

Who is blocked, on what, and **what they do today instead**. The workaround is the most important
fact in this section — if it is tolerable, this feature is optional and should be prioritized as
optional.

Include evidence. "Sales says it is painful" is [reported] and belongs here labeled as such; it is
not the same as a measured volume.

## 3. Why now

What changed. A problem that has been true for three years and is being funded today has a trigger
— name it. If there is no trigger, that is worth knowing.

## 4. Goals

Ranked, not a set. If everything is equally important, no tradeoff can be resolved without me in
the room, and I will not always be in the room.

## 5. Non-goals

What we are deliberately not doing, and why. Include the things stakeholders asked for and were
declined — this is where a PRD earns its keep six months later.

## 6. Users and jobs

Who touches this, and what they are trying to accomplish. Call out where **the person doing the
work is not the person getting the benefit** — that asymmetry is usually the design's hardest
constraint, and it is the one most often discovered late.

## 7. Requirements

| # | Requirement | Priority | Acceptance criteria | Notes |
|---|---|---|---|---|
| R1 | | P0 | | |

Acceptance criteria must be checkable by someone who did not write them. "Works correctly" is not
a criterion.

## 8. What this depends on

Systems, teams, and data that must exist or change first. Sequence infrastructure and schema
changes ahead of application code; sequence shared-library changes ahead of their consumers.

## 9. Data and privacy

What data is read, written, retained, and for how long. Who can see it. If this touches personal
data, the review is a dependency with a ticket, not a footnote.

## 10. AI-specific (delete if not applicable)

- **What the model decides vs. proposes.** Anything that reaches a system of record without human
  confirmation needs its failure mode written out here explicitly.
- **Human-in-the-loop gates.** Where a person must confirm, and what they see when they do.
- **Evaluation.** How we measure quality before launch and continuously after. A model with no
  eval harness is not shippable.
- **Failure behavior.** What the user sees when the model is unavailable, slow, or wrong. Silent
  wrong answers are the worst outcome and the default one.
- **Auditability.** Can we reconstruct why a given output happened, later, on request?
- **Correction signal.** When a human overrides the model, do we capture what they changed? This
  is both the quality measurement and the training data, and it is almost always omitted from v1.

## 11. Success metrics and kill criteria

| Metric | Baseline | Target | Timeframe | How measured |
|---|---|---|---|---|

**Kill criteria:** what result makes us stop. Required. A target with no kill threshold is not a
hypothesis.

## 12. Rollout

Phases, gating criteria between them, and the rollback path. Who is in the first cohort and why
they are the right first cohort.

## 13. Open questions

| # | Question | Owner | Needed by | Blocks |
|---|---|---|---|---|

Open questions stay open here. Unknowns do not get quietly resolved into confident prose — that is
the single most common way a document becomes wrong.

## 14. Decisions log

| Date | Decision | Reasoning | Alternatives rejected |
|---|---|---|---|

Superseded decisions are struck through with a date, never deleted. The reasoning is what makes it
possible to revisit the call correctly when circumstances change.
