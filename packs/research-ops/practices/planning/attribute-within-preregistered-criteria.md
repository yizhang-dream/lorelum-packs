---
{"id":"research-ops.planning.attribute-within-preregistered-criteria","title":"Attribute Results Only Within Pre-Registered Criteria","stage":"planning","tech_stack":["research","ml","agent-ops"],"applies_when":"an experiment produced a negative or partial result and the next planning step is to interpret it, stop a line of work, or fund another route","severity":"warn"}
---

## When to apply

Apply when interpreting results that will change resource allocation - stopping a line, promoting a
mechanism, re-ordering the roadmap - or before extending a local result into a claim about other routes.

## Guidance

State both halves of every verdict: what was tested and what was not. A local negative result refutes only
the mechanism it actually exercised; it does not license "this was the only remaining lever" or "that
problem class is closed".

Give each main experiment exactly one resource decision: before picking arms, data, or gates, write what
will be continued and what will be stopped for each possible outcome. Labeling a changed component is not
a controlled comparison - two arms sharing a new data pool differ only in how they consume it.

Order the evidence: where the current system fails, whether the new mechanism improves those states,
whether the learner absorbs it, whether it converts into end-to-end gains. Separate "the capability was
never produced" from "it was produced but not learned" before changing anything; otherwise every flat
round invites one fresh explanation picked from encoding, curriculum, scheduling, or reward.

Treat pre-registered thresholds as authorization for resource decisions only - ranking configurations on a
small sample, or stopping a degrading run - never as proof that a mechanism works or that a route is
falsified. Keep engineering guardrails at "reliable enough to run the experiment"; one incident does not
automatically escalate into a new round of infrastructure.

Isolate state and data assets by source group: states from one episode, seed, or asset variant stay
together, every train/acceptance split runs on group boundaries, and development material, teacher
confirmation, and student acceptance remain physically separate. Never reuse acceptance states to select
the method for the next variant.

Every batch must retire one important uncertainty. If a run ends with gains unexplained and failures
refuting nothing specific, that is the plan's largest strategic cost - re-sequence so that the question
asked and the material consumed belong to the same batch.

## Why

Over-extension feels cheap because one negative experiment appears to speak for a whole direction, and the
resulting downgrade has no data behind it. Confounded arms and mis-sequenced material compound it: the
project spends compute without shrinking the space of live hypotheses.

## Exceptions and boundaries

A negative result does justify stopping within its scope - the same mechanism, state distribution,
information constraints, and budget. A failure verdict may read "under these conditions the intervention
did not show an advantage worth continuing"; it may not diagnose a capability failure or conclude that
alternative routes are better. Uncontrolled continuation cannot borrow explanatory power from idle
compute: the minimum comparison is an equal-budget control arm from the same starting point and executor.

## Example

A run shows that re-consuming a fixed pool on a different schedule yields no marginal gain on one
subsystem. The draft verdict escalates it to "structural repair is the only lever left" and downgrades an
unrelated route. The corrected verdict records a resource-level stop for that consumption method only:
whether re-collecting states the current system actually visits helps, and whether search can propose
better decisions, were never tested and stay open.
