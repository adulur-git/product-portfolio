# Shipping with AI coding agents

I use Claude Code as part of my actual job — not to prototype ideas, but to make production
changes in the same monorepo my engineers work in. This is what I have learned doing it, including
the parts that are less flattering.

In a recent stretch: **20+ commits merged to `main` across three production services**, 76 commits
authored including in-flight branches, ~11,700 lines. The mix was `feat`, `fix`, `docs`,
`refactor`, and `test` — mostly `fix` and `docs`, which is the honest shape of real work.

---

## Why a PM should do this at all

The usual argument is "technical credibility with engineers." That is a side effect. The real
returns:

**Estimates stop being a negotiation.** When I have read the code a feature touches, I am not
asking an engineer to defend a number to me. Most of the friction in scoping comes from the PM
having no independent model of the cost, so every estimate feels like it might be padded and
every pushback feels like it might be sandbagging.

**I find out what the ask actually costs before I ask.** The most expensive requirements are the
ones that look small from the outside. Being able to go look means I sometimes withdraw a request
before it becomes a ticket, which is the cheapest possible form of scope management.

**Bug reports get better.** I file the failing test, not the description of the symptom.

**I learn where the system is fragile.** Not from an architecture review — from having a change
rejected by a gate I did not know existed.

---

## Where the agent genuinely earns its keep

- **Reading unfamiliar code.** This is the single biggest win and it is not code generation. Point
  it at a subsystem and ask what happens when a specific input arrives. It gets you to the
  relevant 200 lines out of a 400MB repo in a minute.
- **Regression tests.** Describe the bug, get a test that reproduces it. Cheap, high-value, and
  the thing least likely to get done by hand under deadline.
- **Small, well-bounded changes.** UI states, validation, wiring a field through an existing
  layered path. Work with a clear right answer and low design content.
- **Documentation and decision records.** Turning a decision I already made into a durable artifact.
- **CI failure triage.** Reading a red pipeline and telling me whether it is my change, a harness
  problem, or an infrastructure flake — a distinction that used to cost me an engineer's afternoon.

## Where it will hand you something wrong, confidently

**It writes tests that pass against the bug.** The most dangerous failure mode. It will produce a
test, the test will be green, and the test will not actually exercise the broken path. The
discipline: **watch the test fail before you accept it as proof of a fix.** If you never saw red,
you have no evidence.

**It probes the wrong entry point.** If your change wraps an existing function, an agent will often
verify by calling the inner helper — which reproduces the pre-fix behavior by construction and
reports success. Verify through the door the change introduced.

**It grades itself on the wrong signal.** A test command can exit 0 with everything skipped. An
agent will read that exit code as a pass. Read the actual test counts.

**A populated fixture proves nothing about a required field.** It will show you a passing example
and call the contract verified. Check the schema, not the sample.

**It has no idea what matters.** It will fix a typo and a load-bearing business rule with the same
confidence and the same tone. Judgment about which changes are dangerous is entirely yours, and
the agent's fluency actively works against you here — wrong answers arrive sounding exactly like
right ones.

**It will happily widen scope.** Ask for a fix, get a fix plus a refactor plus a renamed variable.
In a shared codebase that is not a bonus, it is a review burden you are imposing on a colleague.

---

## The operating rules I actually follow

1. **Smallest diff that satisfies the requirement.** Before writing new code, check whether it
   already exists in the codebase, whether the language or framework already does it, or whether
   an existing dependency covers it. Most "we need to build X" is "we already have X."
2. **Red before green.** No regression test counts as proof unless I watched it fail first.
3. **Verify at the real entry point.** The one the change introduced, not a convenient inner helper.
4. **Read the counts, not the exit code.**
5. **Never merge my own change on a self-assessment.** The gates exist and they apply to me. Draft
   PR until CI is green, real review, merge queue.
6. **Stay inside the ticket.** If I notice something else, it becomes a separate ticket.
7. **I own the diff.** The agent wrote it; my name is on the commit. "The AI did it" is not a
   defense that exists.

## What I tell the PMs I lead

Learn to read the code before you learn to write it. Reading is where the compounding value is —
it changes how you scope, estimate, and argue. Writing is a smaller, later benefit and carries
real risk of shipping something confidently wrong into a codebase other people maintain.

And do not use this to work around your engineers. The point is to arrive with better questions,
not to hand over a branch and call it a spec.
