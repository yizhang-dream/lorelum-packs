---
{"id":"narrative-writing.delivery.freeze-exit-criteria-before-iterating","title":"Freeze Exit Criteria Before Iterating","stage":"delivery","tech_stack":["writing"],"applies_when":"a narrative project already has review machinery that finds real defects and is still adding review dimensions, axes, or reports instead of finishing drafts","severity":"warn"}
---

## When to apply

Apply when a long-form narrative project's quality system has matured: every review round finds real
problems, yet the draft is no closer to done. Signals include proposals for new audit dimensions or
scoring tables, each fix spawning another pass, and completion that keeps receding.

## Guidance

- Freeze explicit exit criteria in one document before the next iteration: defect counts by severity,
  mechanical checks that must pass, blind-review thresholds, a length band, and no structural
  regression across two consecutive rounds. Meeting them ends the work, residual low-severity
  defects included.
- Stop opening new review documents or dimensions. Route every new mechanism into an existing
  artifact - exit criteria, experience budget, red lines, provenance log - as an added clause, never
  as a new file.
- Spend iteration budget on expansion under the experience budget, then run a light re-check: topic
  grep, mechanical suite, sweep for documents affected by the change. Re-auditing everything each
  round is not the working mode.
- Ship with bounded known debt: record each accepted defect, cap the count, and let delivery proceed.
- Admit new material by function. A researched detail or lore addition must serve plot, character,
  or humor, or it stays out of the draft.

## Why

Review axes with high recall and low termination are unbounded by construction - each one can always
find something. When the goal becomes "no defect anywhere", quality rises while distance to
completion grows with it. Frozen thresholds plus a bounded debt allowance turn an endless search for
perfection into a finish line the team can actually cross.

## Exceptions and boundaries

A debt allowance never covers severe defects: canon contradictions, unusable deliverables, or content
that breaks the promised rating still block. Exit criteria also do not forbid fixing a severe defect
found late; they cap iteration on low-severity polish, not severity itself. Freezing criteria assumes
detection already works - if the review machinery misses real problems, fix detection first.

## Example

A narrative team's review stack could already detect most defects, and two more audit dimensions were
proposed in one week. The owner froze an exit checklist - zero top-severity issues, at most five
low-severity ones, the mechanical suite green, and two consecutive rounds without structural
regression - and required that every new mechanism become a clause in an existing document. The work
finished on that checklist with a handful of recorded, deliberately accepted defects.
