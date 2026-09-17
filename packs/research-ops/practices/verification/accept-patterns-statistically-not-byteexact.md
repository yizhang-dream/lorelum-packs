---
{"id":"research-ops.verification.accept-patterns-statistically-not-byteexact","title":"Accept Patterns Statistically, Not Byte-Exact","stage":"verification","tech_stack":["research","simulation"],"applies_when":"an emergent or noisy simulation cannot be validated by reproducing an exact reference output, and the acceptance criterion must still be objective and preregistered","severity":"warn"}
---

## When to apply

Apply to systems whose outputs are emergent, stochastic, or hardware-dependent - agent
simulations, chemistry worlds, ecology models - where a golden-output diff either does not
exist or fails for reasons that say nothing about correctness.

## Guidance

Define acceptance as a battery of independent, measurable patterns rather than a byte-exact
baseline. Each pattern is a statistic over run output: a distribution (step lengths, turn
angles, cluster sizes), a conservation residual, a rate, or a structural count. For every
pattern, preregister the observable, the sampling window, and the tolerance, then evaluate all
of them on production runs. A run passes only when every pattern stays inside tolerance.

Keep the patterns independent of one another - one from physics, one from chemistry, one from
population structure - so that agreement across several unrelated statistics is real evidence
rather than one metric counted twice. Retain exact reproduction where it applies: event
ordering, seeded reproducibility, and deterministic components can still be compared bit for
bit even when the aggregate output cannot.

## Why

Noisy systems fail byte comparisons constantly for legitimate reasons (floating-point order,
scheduling, hardware), so a byte gate degenerates into a permanently red light that everyone
learns to waive. Patterns convert genuine failure modes into quantitative statements with
tolerances while staying objective, and requiring several independent patterns to agree
prevents the failure mode of single-metric gating, where one easy statistic masks a real
regression.

## Exceptions and boundaries

Tolerances must come from analyzed baseline runs, not from what makes the current run pass; a
tolerance chosen after seeing the result is not a gate. Statistical acceptance does not replace
engineering tests of deterministic paths - keep bit-level checks where output is supposed to
be bit-identical, and never widen a tolerance without a preregistered reason.

## Example

An emergent-world simulator replaced golden-output comparison with a pattern battery:
per-stage patterns from diversity counts to autocatalysis ranks and conservation identities,
each with a preregistered tolerance and sampling window. Determinism stayed a bit-level
requirement only for the event stream, so refactors could be verified exactly while the
science was judged by the battery.
