---
{"id":"research-ops.testing.design-tests-around-p-value-floors","title":"Design Tests Around p-Value Floors","stage":"testing","tech_stack":["research","simulation"],"applies_when":"a preregistered comparison uses an exact permutation or rank-based test on small group sizes, and the significance threshold is near the smallest p-value the design can produce","severity":"warn"}
---

## When to apply

Apply whenever a statistical gate is preregistered with a fixed threshold and the test is
exact: permutation tests, sign tests, or any procedure whose p-value can only take values
determined by the sample size.

## Guidance

Compute the smallest attainable p-value from the design before promising a threshold. For an
exact two-group permutation test with k items per group, the most extreme possible split gives
p = 1/C(2k,k): 0.05 at k=3 per group, 0.00029 at k=7. Choose group sizes so the floor sits
strictly below the threshold with room to spare, and state the floor in the preregistration
next to the threshold, so a later "test failed" cannot be confused with "this test could never
pass".

Where an effect is real but the sample too small, the fix is more units or more trials, not a
looser threshold. If the floor is already barely below the threshold, treat any pass as
fragile and require an independent replicated run before it counts as evidence.

## Why

An effect can be genuine and still be unreachable: when threshold and floor coincide, even a
perfect separation yields exactly p equal to the threshold, which a strict comparison fails.
That outcome is a property of the design, not evidence about the world, and it is invisible in
the p-value alone - only the design math reveals it. Discovering it after the run wastes the
compute and, worse, invites quietly relaxing the gate.

## Exceptions and boundaries

Asymptotic tests (t-test, chi-square) have no such discrete floor, but they replace it with
distributional assumptions that exact tests deliberately avoid; the floor argument applies
only to exactness. For weighted, paired, or multi-group designs the floor is a different
combinatorial quantity - derive it for the actual statistic instead of copying the two-group
formula.

## Example

A preregistered compositional gate used an exact permutation test with 3 items per group. It
returned 1/C(6,3) = 0.05 against a 0.05 threshold - a structural tie no data could break. The
revised preregistration moved to 7 per group (floor 1/C(14,7) = 0.00029) and the rerun passed
at p < 0.001, converting a design failure into a clean, interpretable result.
