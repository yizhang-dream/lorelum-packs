---
{"id":"research-ops.verification.grade-negative-results-by-confidence","title":"Grade Negative Results by Confidence Before Citing Them","stage":"verification","tech_stack":["research","ml"],"applies_when":"a negative or failed result is about to be used in a conclusion, a plan, or a decision, and the underlying evidence comes from runs at limited scale, an older recipe, or with unfactored confounds","severity":"warn"}
---

## When to apply

Apply whenever a failure is quoted as a reason to abandon a direction, to justify a redesign, or to
argue that a capability is unreachable. Also apply to the mirror case: a surprisingly easy positive
result that is about to be reported without caveats.

## Guidance

Attach a scope qualifier to every negative claim: the scale actually run (a few dozen parallel
environments versus thousands), the recipe or configuration version, and whether known confounds were
removed. Then grade confidence explicitly:

- High: known bugs were controlled for, the mechanism is explainable, and multiple variants reproduce
  the failure.
- Medium: one controlled failure with an explanatory account.
- Unknown-bug territory: a single failure with no causal story - state this honestly; an unknown bug
  cannot be logically excluded, so the result cannot be packaged as a denial.

Before attributing any failure to mechanism or tuning, verify metric identity in code. A logged key
whose name resembles a familiar quantity may be a different quantity, a configured value may never be
executed, and an aggregate may mix units. Diagnose implementation defects before opening a
hyperparameter menu. Prefer low-cost, high-information checks over large reruns, and when a mechanism
hypothesis is proposed, answer it with a minimal discriminating design rather than another broad
sweep.

Apply the same treatment to unexpectedly good results: state which pipeline was actually measured
(official recorded inputs replayed through the executor is not the same chain as an end-to-end
system), and list the open caveats instead of celebrating.

## Why

Without scope, an engineering boundary is read as a scientific conclusion, and a direction may be
abandoned for a limit that belonged to one configuration at one scale. Without code-level identity
verification, remediation menus are built on narrative mismatch - the fix that follows from "the
update is too hot" is wrong if the logged quantity was never the update magnitude. Graded confidence
keeps the project's own record honest and makes future reviews cheaper.

## Exceptions and boundaries

When a result is broad, representative, and reproduced across configurations, state it plainly and stop
hedging it. Do not use "there could be a bug" as a rhetorical shield after bugs have been excluded;
that is a different failure of honesty. Do not convert the grading into permission to ignore a
well-scoped failure - a limited negative result is still a negative result within its stated box.

## Example

A decoder-tuning claim of "all variants fail" becomes "all variants failed at the scale tested under
that year's regularisation recipe; larger-scale retraining was never run". In a separate review, a
proposed "learning rate too hot" fix menu is withdrawn after a line-by-line read shows the logged
quantity was a divergence to a fixed reference rather than the policy update, and a distinct value in
the config was never consumed by the training loop.
