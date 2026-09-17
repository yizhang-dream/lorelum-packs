---
{"id":"research-ops.testing.benchmark-with-clean-invalid-sample-policy","title":"Benchmark With a Clean Invalid-Sample Policy","stage":"testing","tech_stack":["research","ml"],"applies_when":"a multi-suite capability benchmark will be compared across models, versions, or configurations, and some samples are expected to fail for infrastructure rather than capability reasons","severity":"warn"}
---

## When to apply

Apply when building or running a benchmark whose suite scores will drive decisions - model
selection, version comparison, regression tracking - and where rate limits, truncated
generations, or sandbox errors are normal operating conditions.

## Guidance

Fix the invalid-sample policy before scoring begins. Distinguish truncated samples (budget
exhausted) from failed samples (rate limits, transport, sandbox errors), and require failed
samples to be re-run rather than dropped: when errors correlate with the hardest items - as
they do, since long reasoning burns budget and hits limits - "count only valid items" silently
deletes exactly the cases that would score low. Keep per-item records so any aggregate can be
recomputed.

Handle budget exhaustion with a preregistered escalation ladder: re-run truncated items at
larger budgets step by step, and make each escalation robust to nondeterministic
infrastructure by probing repeatedly before accepting any failure as final. Verify each
suite's scoring independently - answer extractors, symbolic equivalence, format validators -
because a scoring bug shifts all results in one direction and stays invisible in a score
table. Finally, check saturation: when a suite's scores cluster at the ceiling for the
strongest models it has stopped discriminating; report it without using it to rank.

## Why

An invalid-sample policy that hides its own bias produces rankings that look precise and are
wrong. Selectively dropping errors rewards whatever fails most on hard items, while re-running
them costs compute but keeps the comparison honest. Escalation ladders and independent scoring
checks protect the two other silent failure modes: an infrastructure hiccup scored as a model
failure, and a parser bug that quietly compresses everyone's scores.

## Exceptions and boundaries

Saturated suites can still serve as coverage or smoke checks; they simply carry no ranking
signal. Re-running is not always possible under fixed API quotas - record what was dropped and
why, and report the affected suites as lower-confidence instead of quietly ranking on them.

## Example

A six-suite model comparison kept per-item records for thousands of questions. Two policies
prevented a false ranking: invalid samples were re-run rather than excluded, which block the
alternative that would have fabricated perfect scores on a suite whose errors concentrated on
hard items; and truncated items escalated through a budget ladder with repeated probes before
counting as failures. One suite had saturated at the ceiling and was reported without rank
claims, while a suite with real spread served as the discriminator.
