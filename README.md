# Product Portfolio — Alekhya Dulur

Principal Product Manager. Enterprise platforms, revenue systems, and agentic AI workflows.
Profile and contact: **[github.com/adulur-git](https://github.com/adulur-git)**

This repository is the long-form version of my resume: the actual reasoning behind a few
decisions, the tradeoffs I took, and the ones I got wrong.

---

## Contents

**PRDs** — how I actually write a spec.

- [PRD template](prds/TEMPLATE.md) — the structure I use, including the sections people skip
- [Worked example: unified inbound triage](prds/example-inbound-triage.md) — a complete PRD on a
  hypothetical product, so the craft is visible without disclosing anything

**How I work** — the operating manual.

- [Working with engineering](how-i-work/working-with-engineering.md)
- [Shipping with AI coding agents](how-i-work/shipping-with-ai-agents.md)

---

## Evidence standard

Because AI makes it very cheap to write a confident, wrong document, I hold my own artifacts to a
labeling rule. Every load-bearing claim here is one of:

- **[measured]** — I have the query or dashboard behind it
- **[reported]** — someone credible told me, and I name who
- **[inferred]** — a reasonable read, not a fact
- **[unknown]** — an open question, stated as open

Numbers with no bottom-up derivation stay labeled as projections. If a claim later turns out
wrong, I supersede it in place with a date rather than quietly deleting it.

---

## What I look for in a product problem

1. **Who is actually blocked, and what do they do today instead?** If the workaround is fine, the
   feature is optional.
2. **What is the smallest thing that changes behavior?** Not the smallest thing that ships.
3. **What does this cost the people who have to maintain it?** Every integration is a permanent
   liability, and a PM who ignores that is expensive.
4. **How will we know it worked, and what result would make us stop?** A success metric with no
   matching kill criterion is marketing.
5. **What are we explicitly not doing?** Non-goals are the most useful section of any PRD.
