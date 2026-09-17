---
{"id":"research-ops.planning.separate-gates-from-mechanisms","title":"Separate Gate Outcomes From Mechanism Claims","stage":"planning","tech_stack":["research","ml"],"applies_when":"designing or reviewing an experiment whose result will be reported as a pass or fail against a threshold, or writing the pre-registration for the next round after such a gate has fired","severity":"warn"}
---

## When to apply

Apply at two moments: when a pre-registration is being written and the output format is still being
chosen, and when a gate has fired and the result is being narrated into a conclusion. The second is
where the damage usually happens - a channel-level pass is narrated as a mechanism finding, and the
next round loses its identification power.

## Guidance

Keep the two senses of pass apart in wording and in the report structure:

- PASS-AS-CHANNEL: the pathway can carry a usable signal and the effect is repeatable.
- PASS-AS-MECHANISM: causal identification has been achieved.

Falsification (ruling out competing explanations) and identification (locating the cause) are separate
stages; never merge them into one sentence. Hard thresholds serve as gates only, not as a scientific
explanation of an effect size, and the decay structure of an effect is itself an object to explain.

Pre-register before launching: parameter and grid choices with reasons, conditioning variables whose
mapping is frozen before evaluation, the primary metric, and a stopping rule that triggers
consolidation instead of point-by-point significance hunting. Declare the output as effect curve plus
uncertainty plus conditioned effects - not a single point pass. Fix the execution order: specification,
pre-registration freeze, implementation, dry-run and integrity checks, then compute; compute
availability is not a scientific reason to start.

Cap the number of rules: once the design is frozen, switch from "can this be stricter" to an operator
test - whether a stranger can execute it unambiguously from a short document. Do not loosen a
conservative stopping rule after seeing results, report non-significance against a pre-registered
practical-null boundary rather than as zero, and treat a freeze as a pure state transition: changing a
treatment mapping or adding an estimand after freeze invalidates the experimental identity and
requires re-registration.

## Why

A gate outcome narrated as a mechanism makes the next experiment a point-accumulation exercise: with
the mechanism assumed, the only remaining question is more grid points, and the design stops being able
to discriminate. Thresholds used as explanations also smuggle arbitrariness into the interpretation,
because a decision boundary is not a natural constant. Separating the layers is what keeps a negative
or partial result informative.

## Exceptions and boundaries

Exploratory localization produced by a closed experiment may guide the next design but never counts as
confirmatory evidence, and should be labelled as such. A conditional pass must not be written as a full
pass; a failed stage does not by itself establish that the target is unreachable. If a mechanism claim
is wanted, it requires a designed discriminating test, not a denser grid.

## Example

A controller variant passes its threshold, and the draft conclusion says the mechanism works and only
needs a finer grid. The review corrects it to a channel-level pass, keeps the threshold as a gate, and
requires the next round to pre-register an effect curve, uncertainty, and a stopping rule before any
runs are launched.
