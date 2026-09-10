# Working with engineering

My operating manual. Written down so people can hold me to it, and so the PMs I lead have
something concrete to disagree with.

---

## What I owe engineering

**A problem, and the constraints — not a solution.** If I have specified the implementation, I have
both exceeded my competence and wasted theirs. Where I do have an implementation opinion, I say so
explicitly and label it as an opinion, so it can be overruled with data.

**Explicit non-goals.** The most useful section of any PRD. Unbounded scope is a PM failure, not an
engineering one.

**One prioritized list, and the courage to keep it prioritized.** Three P0s means no P0s. When
something new is genuinely urgent, I say what it displaces in the same sentence.

**A decision, when a decision is what is blocking.** Engineers stalled on an unanswered product
question is the most expensive failure mode I can personally cause. If I need a day, I say it is a
day.

**Real context on the why.** Not motivational framing — the actual business mechanism. Engineers
make dozens of small judgment calls I will never see, and they make them well only if they
understand what the thing is for.

**Protection from thrash.** Requirements moving mid-sprint because I did not do my discovery is my
bill to pay, not theirs.

## What I ask for

**Tell me the cost early, especially when it is bad news.** I would much rather hear "this is three
weeks, not three days" on day one. I can requote a stakeholder. I cannot un-miss a date.

**Push back on the requirement, not just the estimate.** If the ask is wrong, the useful response
is "this will not solve the problem you described," not a padded estimate.

**Tell me when I am the bottleneck.** I will not notice on my own.

**Name the maintenance cost.** I am systematically biased toward things that look cheap to build. If
a design is fast now and a permanent liability later, that tradeoff needs to be on the table where
I can see it.

---

## Practices

**Verify the ticket against the code before committing to it.** A ticket is a claim made on the day
it was written. Requirements go stale, decisions land, someone builds half of it in passing. Before
estimating, I check merged code, open PRs, and any decision records since. I have killed tickets
this way that were already done.

**Decisions get written down where they will be found.** A decision that lives only in a meeting
gets re-litigated in a quarter by people who were not there. Short, dated, with the reasoning and
the alternatives rejected — the reasoning is what makes it possible to revisit correctly when
circumstances change.

**Decompose without losing scope.** Fewer tickets is not less work. When I break something down,
every acceptance criterion in the original has a named home in a child, or it was deliberately cut
and recorded as cut. Silent scope loss during decomposition is the failure I watch for hardest.

**Infrastructure and schema work sequences first.** Data model and infrastructure changes land ahead
of the application code that depends on them, and shared-library changes land ahead of their
consumers. Getting this order wrong produces a sprint where everything is 90% done and nothing
works.

**Kill criteria, not just success criteria.** Before we build, what result would make us stop? A
metric with no matching kill threshold is not a hypothesis, it is a press release.

**I read the PRs on my product.** Not to approve them — to know what actually shipped versus what
the ticket said.
