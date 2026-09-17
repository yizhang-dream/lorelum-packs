---
{"id":"research-ops.review.rotate-reviewers-and-date-decisions","title":"Rotate Review Lenses and Date Decision Records","stage":"review","tech_stack":["research","agent-ops"],"applies_when":"a multi-round review or validation effort is running and verdicts, thresholds, or claim wording change between rounds without a durable record of which verdict supersedes which","severity":"warn"}
---

## When to apply

Apply when an effort depends on external review across several rounds: pre-registration, validation
frameworks, safety or governance claims, or any process where each round may revise earlier findings
and where the final claim must be defensible later. Also apply whenever a gate is described as
enforced by an automated check.

## Guidance

Run each round with a different review lens rather than repeating one checklist. Typical rotation:
engineering soundness, data governance, measurement quality, pre-registration completeness,
statistical method, and the identity of the construct or claim itself. Give each lens its own
lettered finding series so citations such as a finding number identify exactly one lens.

Record every round as a dated decision document containing the lens, the numbered findings, and an
explicit verdict per area. Then:

- Treat a conditional pass as not a pass. Record verdicts such as not ready to freeze honestly,
  rather than rounding them up.
- When a later round replaces an earlier verdict, keep the old text and add a superseded-by pointer
  to the new document. Never delete or silently edit a past verdict.
- Bump the artifact or product version only for substantive rounds; documentation-only rounds do
  not raise it.
- Before citing a check script, test file, or tool as evidence of a gate, confirm it exists at the
  stated path and run it. If it is missing, perform the equivalent check manually and say in the
  record that the automated gate was absent.

## Why

A reviewer who repeats the same lens converges on the same blind spots; rotating lenses forces each
dimension to be examined on its own terms. Dated, numbered records survive context loss between
sessions and make it possible to tell which rule is currently in force. Supersede pointers preserve
the audit trail instead of replacing it. A gate that references a script which does not exist is
not a gate, and claiming it as automated evidence is worse than admitting it is manual.

## Exceptions and boundaries

A small, single-dimension change does not need a full rotation cycle. Keep the reviewer independent
of the implementation - a different person or a separate session - wherever the credibility of the
result depends on it. Rotation does not mean ignoring repeated findings: an open finding must
either be resolved or explicitly accepted as debt with a boundary.

## Example

A validation effort runs six rounds, each with a distinct lens and its own lettered findings. Each
verdict lands in a dated decision document; when a later construct review supersedes an earlier
naming claim, the old document gains a pointer to the newer one and both remain readable. A script
cited as a documentation-tree gate turns out not to exist, so the team performs the equivalent
manual check and records that the automated form was never present.
