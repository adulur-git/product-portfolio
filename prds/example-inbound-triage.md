# PRD: Unified inbound triage

**Owner:** Alekhya Dulur · **Status:** Worked example
**Last updated:** 2026-09-09 · **Engineering partner:** — · **Design partner:** —

> **This is a writing sample, not a real product.** The scenario is hypothetical and all figures
> are illustrative, labeled **[illustrative]**. It exists so the craft is visible without
> disclosing anything from an employer. Structure follows [TEMPLATE.md](TEMPLATE.md).

---

## 1. Summary

Inbound requests arrive across four channels and are triaged by hand into the queue that owns
them. We are building an agent that reads each request, proposes a routing decision and a
structured summary, and asks a human to confirm. The goal is to cut time-to-first-touch and to make
inbound volume measurable for the first time.

## 2. Problem

A shared inbox, a web form, a Slack channel, and forwarded email all feed the same twelve people.
Each request is read by a coordinator who decides which of six queues owns it and retypes the
relevant details into the tracking system.

**What they do today instead:** the coordinator rotation. It works. That is important — this is not
a broken process, it is an expensive one, and that changes how aggressively we should ship.

- Median time-to-first-touch: **6.5 hours**, p90 **31 hours** [illustrative, measured]
- Requests arriving outside the coordinator's timezone wait until morning [inferred]
- Misroutes requiring a second hop: **~18%** of volume [illustrative, measured]
- Coordinator load: roughly **1.5 FTE** across the rotation [illustrative, reported — ops lead]
- Requests never entered into the tracker at all: **[unknown]**. We believe it is non-trivial for
  the Slack channel, and we cannot currently measure it. This is the number I most want.

The misroute rate is the sharpest cost: a misrouted request does not just wait, it waits *twice*,
and the requester usually has to re-explain.

## 3. Why now

Two triggers. Inbound volume is up **~40% year over year** [illustrative, measured] against a flat
coordinator rotation, so the manual path is nearing its ceiling. And extraction quality from
general-purpose models crossed the threshold where confirmation is cheaper than authoring — that
was not true when this was last considered.

## 4. Goals

1. Reduce median time-to-first-touch
2. Reduce misroutes
3. Make total inbound volume measurable, including what never gets entered today
4. Reduce coordinator toil

Ranked deliberately. If a design choice trades measurement for a marginal speed gain, take the
measurement — we can improve what we can see, and (3) is the goal that unlocks the next two years
of decisions.

## 5. Non-goals

- **Auto-responding to requesters.** Out of scope entirely. Small upside, and a wrong automated
  reply to a customer is a much worse day than a slow correct one.
- **Replacing the coordinator role.** The agent proposes; a human confirms. We are removing
  retyping, not judgment.
- **Priority scoring.** Asked for by two stakeholders. Declined for v1: a ranking model built on
  our current data would learn our current blind spots, and we would not be able to tell.
- **Consolidating the four channels into one.** The right long-term answer and a separate,
  political, much larger project. Requiring it as a precondition would sink this one.
- **Historical backfill.** No decision is waiting on it.

## 6. Users and jobs

| User | Job | What they care about |
|---|---|---|
| Coordinator | Get each request to the right queue, fast | Not retyping; not being blamed for misroutes |
| Queue owner | Receive requests that are actually theirs, with context | Fewer bad handoffs |
| Requester | Get an answer | Speed; not re-explaining |
| Ops lead | Understand volume and staffing | Trustworthy numbers |

**The asymmetry that drives the design:** the coordinator does the work and the ops lead gets the
benefit. Every second of added coordinator friction is paid by someone who was not asking for
this. The interaction has to be *faster than what they do today from the first day*, or adoption
fails and we learn nothing.

## 7. Requirements

| # | Requirement | Priority | Acceptance criteria | Notes |
|---|---|---|---|---|
| R1 | Ingest from all four channels | P0 | A request submitted through each channel appears in the triage queue within 2 min, verified once per channel | Slack is the hard one — no structured envelope |
| R2 | Propose a routing decision with confidence | P0 | Every queued item carries exactly one proposed queue and a confidence value | |
| R3 | Propose a structured summary | P0 | Requester, request type, urgency, and free-text summary populated or explicitly marked unextractable | "Unextractable" is a valid, useful output — never guess |
| R4 | Human confirm or correct in one action | P0 | Accepting an unmodified proposal is a single click; correcting the queue is one dropdown + confirm | If this is slower than today, we have failed |
| R5 | Capture the correction diff | P0 | Every human change records field, prior value, new value, timestamp | Both our quality metric and our training data. Cheap now, unrecoverable later |
| R6 | Deduplicate across channels | P1 | A request sent to the inbox and the Slack channel within 1h surfaces as one item with both sources | Common, and irritating when missed |
| R7 | Route on confirm | P0 | Confirmed item creates the tracker record and notifies the queue | |
| R8 | Full audit trail | P0 | For any routed request, reconstruct the original artifact, the proposal, the confirmer, and the diff | |
| R9 | Degrade to manual | P0 | With the model unavailable, items still queue with empty proposals and remain triageable | The system must never be a single point of failure for inbound |
| R10 | Volume dashboard | P1 | Daily volume by channel, type, queue; misroute and correction rates | This is goal 3 |

## 8. What this depends on

- Read access to all four channels — the Slack app needs a scope approval, **lead time [unknown]**,
  and this is the most likely schedule risk
- Tracker write API — exists; rate limits unverified
- Queue taxonomy — the six queues are stable per the ops lead [reported]. If a reorg is coming,
  we should know before we encode it

Sequencing: schema and ingestion land before the model work. The correction-diff store (R5) ships
with the first confirmable proposal, not after — retrofitting it means throwing away the early
data, which is the most valuable data.

## 9. Data and privacy

Inbound requests contain requester identity and may contain customer-confidential detail. Nothing
new is collected — this is a new store of data we already hold, which is still a review trigger,
not an exemption. Retention matches the existing tracker. Access is limited to coordinators, queue
owners, and the ops lead. Privacy review is a dependency with a ticket, not a footnote.

## 10. AI-specific

**Decides vs. proposes.** The agent only ever proposes. Nothing reaches the tracker without a human
confirming. Non-negotiable for v1 — the failure mode of a silently misrouted, silently
mis-summarized request is that it looks handled and is not, and nobody finds out until the
requester escalates.

**Human-in-the-loop.** The confirmation screen shows the original artifact side by side with every
proposed field. The coordinator can accept in one action or correct any field. Confidence is shown
per field, not per request — routing and urgency have genuinely different reliability and pooling
them hides that.

**Evaluation.** A labeled set of 500 historical requests with known-correct queues [illustrative],
held out. Pre-launch bar: routing accuracy above the human misroute rate of ~18%. Post-launch, the
correction rate from R5 *is* the continuous eval — that is why R5 is P0.

**Failure behavior.** Model unavailable: items queue with empty proposals (R9). Low confidence:
proposal is shown as low-confidence rather than suppressed — a weak proposal a human can reject is
still faster than a blank form. Never fabricate a field to look complete; "unextractable" is a
first-class output.

**Auditability.** R8. For any routed request we can reconstruct the artifact, the proposal, the
confirmer, and what they changed.

**Correction signal.** R5. Stated separately because it is the requirement most likely to be cut
under schedule pressure and the one that is genuinely unrecoverable if cut.

## 11. Success metrics and kill criteria

| Metric | Baseline | Target | Timeframe | How measured |
|---|---|---|---|---|
| Median time-to-first-touch | 6.5h [illustrative] | < 2h | 90 days post-GA | Timestamp delta |
| p90 time-to-first-touch | 31h [illustrative] | < 8h | 90 days | Timestamp delta |
| Misroute rate | ~18% [illustrative] | < 8% | 90 days | Second-hop count |
| Routing proposals accepted unmodified | n/a | > 70% | 60 days | R5 diff store |
| Coordinator time per request | [unknown] | -50% | 90 days | Timed sample, pre and post |
| Total measured inbound | [unknown] | Established | 30 days | R10 dashboard |

**Kill criteria:**

- Accepted-unmodified below **50%** at 60 days. Below that, confirmation is not cheaper than
  authoring and the core premise is wrong.
- Coordinator time per request **not lower** than baseline at 90 days. We would be taxing the user
  who does the work to benefit someone else — the exact failure the goal ranking was written to
  prevent.
- Misroute rate **at or above** baseline at 90 days.

Any one of these triggers a stop-and-reassess, not a rescope. I want that written down now, while
nobody is invested.

## 12. Rollout

1. **Shadow (2 weeks).** Agent proposes; coordinators never see it. Compare proposals to their
   real decisions. Gate: routing accuracy beats baseline on live traffic, not just the held-out set.
2. **One channel, opt-in (3 weeks).** Web form only — most structured, lowest risk. Two volunteer
   coordinators. Gate: accepted-unmodified > 60% and no coordinator reports it as slower.
3. **All channels, all coordinators (4 weeks).** Manual path stays fully available.
4. **GA.** Default path; manual remains permanently available per R9.

**Rollback:** a single flag returns every channel to the manual queue. Because we never removed the
manual path, rollback is a config change, not a migration.

## 13. Open questions

| # | Question | Owner | Needed by | Blocks |
|---|---|---|---|---|
| Q1 | How many Slack requests never reach the tracker today? | Ops lead | Shadow phase | Sizing goal 3 |
| Q2 | Slack scope approval lead time? | Me | Before commit | R1, whole schedule |
| Q3 | Is the six-queue taxonomy stable through next FY? | Ops lead | Before build | R2, eval set |
| Q4 | Tracker API rate limits at 40% volume growth? | Eng | Before build | R7 |
| Q5 | Confidence threshold for low-confidence display? | Me + eng | Phase 2 | R4 |
| Q6 | Who owns the eval set as the taxonomy evolves? | [unknown] | Before GA | Long-term quality |

Q6 has no owner and that is the honest state. An eval set with no maintainer decays silently, and
I do not yet know whose job this is.

## 14. Decisions log

| Date | Decision | Reasoning | Alternatives rejected |
|---|---|---|---|
| 2026-09-09 | Propose-and-confirm, never auto-route | Silent misroute looks handled and is not; discovered only on escalation | Auto-route above a confidence threshold — revisit once R5 data exists |
| 2026-09-09 | Per-field confidence, not per-request | Routing and urgency have different reliability; pooling hides it | Single request-level score |
| 2026-09-09 | R5 correction capture is P0 | It is the continuous eval and the training data; unrecoverable if deferred | Ship v1 without it, add in v2 |
| 2026-09-09 | No priority scoring in v1 | Would learn our current blind spots with no way to detect it | Include a simple heuristic scorer |
| 2026-09-09 | Do not consolidate channels first | Correct long-term, but a much larger political project; as a precondition it sinks this one | Block on consolidation |
| 2026-09-09 | Web form is the phase-2 channel | Most structured, lowest risk, fastest clean signal | Start with the shared inbox (highest volume, messiest) |
