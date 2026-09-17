---
{"id":"research-ops.testing.evaluate-coevolution-with-opponent-pools","title":"Evaluate Coevolution With Opponent Pools","stage":"testing","tech_stack":["research","ml","simulation"],"applies_when":"an evolutionary or self-play loop is scored by playing its population against itself, and the resulting win-rate curve will be used to justify further search","severity":"warn"}
---

## When to apply

Apply to any coevolution or self-play evaluation - game agents, negotiation policies,
adversarial populations - where fitness comes from within-population matches and a rising
win-rate curve is the headline evidence of progress.

## Guidance

Build three fixtures before trusting any fitness number. First, an opponent pool: evaluate
each candidate against a sampled, persisted set of opponents rather than only the current
generation, so fitness stays comparable across generations. Second, dedicated exploiter
individuals rewarded purely for beating current champions, so a policy that overfits to a
narrow opponent mix is exposed. Third, a frozen anchor pool of fixed baselines that never
evolves; report win rates against those anchors as the absolute progress measure.

Score results in buckets, not one aggregate: group matches by opponent archetype or capability
tier and report per-bucket outcome distributions. Cross-generation averages hide the cycling
and non-transitivity that coevolution produces, while bucket-level tables make them visible.

## Why

Coevolution makes fitness relative: an agent improves only by beating whoever is currently in
the pool, which invites Red Queen dynamics where measured strength rises while absolute
competence stalls, and cycling where A beats B, B beats C, and C beats A. Anchor pools break
the relativism, exploiters convert latent weaknesses into measurable losses, and bucketed
reporting keeps a single aggregate from averaging away the structure that matters.

## Exceptions and boundaries

Anchors must be strong enough to discriminate - a baseline set that everything beats provides
no signal - and frozen, since silently updating them resets the scale. Fixed-opponent
training, with no coevolution between populations, needs only held-out evaluation, not pool
machinery; the fixtures above are requirements of coevolution itself.

## Example

A design ready to start an evolutionary agent loop was revised before its first run: the loop
specification listed an opponent pool, a small set of exploiter individuals, a frozen anchor
pool as the absolute-strength reference, and bucketed evaluation guarding against
non-transitive outcomes. Reporting within-generation win rates alone was rejected as
uninterpretable before any compute was spent.
