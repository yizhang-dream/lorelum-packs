---
{"id":"research-ops.planning.upgrade-representation-only-on-saturation","title":"Upgrade Representation Only On Measured Saturation","stage":"planning","tech_stack":["research","ml"],"applies_when":"a research line is selecting its agent or method representation and someone proposes a more expressive class (neural networks, deep RL, LLM policies) before the cheap representation has been measured to saturate the primary metric","severity":"warn"}
---

## When to apply

Apply at representation-selection time for any learning or search-based research line - agent
policies, controllers, fitness models - and whenever a plan adds a higher-capacity method "to be
safe" or "in case the simple one plateaus". Also apply when a milestone plan would run two
representation classes in parallel without one being defined as the trigger-gated upgrade.

## Guidance

Choose the cheapest representational class that can express the task, sized to the search problem:
parameterized genomes or feature-based policies fit when the decision structure is low-dimensional
(single-digit to low-double-digit parameters often sit in the sweet spot of derivative-free
optimizers such as CMA-ES).

Write the upgrade trigger down before starting: (1) the current representation has measurably
saturated the primary metric under repeated seeded evaluation, and (2) the residual failures are
attributable to representational capacity - typically nonlinear interactions the current class
cannot express. Both conditions, or the upgrade waits.

Keep the higher-capacity class as a documented branch, not the mainline. Retain the cheap line as a
control arm and sparring population after an upgrade; do not delete it, since it anchors whether a
gain came from capacity or from noise. One experiment line per milestone.

## Why

Representation upgrades are not incremental: tuning cost, compute per sample, and opaque failure
modes all jump, and attribution degrades - when the stronger class succeeds you cannot tell whether
capacity, hyperparameters, or an environment change produced the result. A pre-declared saturation
trigger turns the upgrade into an experiment with a hypothesis instead of an act of faith, and a
measured plateau (not "it feels stuck") keeps the decision reviewable.

## Exceptions and boundaries

An a-priori inadequate representation, or an external baseline you must match, justifies starting
with the expressive class - but still pre-register the metric and the reason. Saturation may not be
asserted from a handful of runs; it needs the same seeded weighting the main line uses.
Representation choice is not the same as implementation tuning: optimizing the cheap class before
upgrading is in scope, rewriting the harness to make the upgrade easier is not.

## Example

A plan proposes small neural policies alongside a parameterized genome "just in case the genome
plateaus". The revision freezes the genome as the mainline, defines saturation as win rate
flattening under repeated seeded evaluation plus a failure mode that requires nonlinear identity
inference, and keeps the neural branch behind that trigger. The milestone now has one line of
attack and a falsifiable reason to escalate.
