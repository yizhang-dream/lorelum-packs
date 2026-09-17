---
{"id":"research-ops.requirements.scope-ai-help-by-integrity-policy","title":"Scope AI Help by the Evaluation Integrity Policy","stage":"requirements","tech_stack":["research"],"applies_when":"an agent is asked to assist with graded, evaluated, or certifying work (assignments, assessments, submissions) and the assistance could amount to authoring part of the submitted artifact","severity":"warn"}
---

## When to apply

Apply at the start of any engagement with evaluated work, and again at every deliverable boundary
inside it. The policy is a requirements source: it defines the boundary of legitimate assistance
before any writing, coding, or explanation happens.

## Guidance

Read the evaluation's integrity policy first and record three things: what it forbids (for example,
obviously machine-authored reports or code), what it requires (disclosure, citation of assistance),
and the penalty if crossed - which can void the whole deliverable rather than a few marks.

Unless the requester explicitly decides otherwise after being told the constraint, keep the default
scope of help to setup, environment repair, verification of the candidate's own output, and
explanation of concepts. Do not author the graded answer text or solution code. When a helper
artifact is needed, reuse the utilities the evaluation itself provides or mandates - substituting a
custom equivalent can lose marks even when it is functionally correct - and keep the requester as
the author of record for the submission.

Re-check the constraint each time the deliverable changes: the policy applies to every submission in
a sequence, and later work builds on earlier reviewed output. If the intended scope is ambiguous,
ask the requester to decide and record the decision before proceeding; never agree to conceal
assistance.

## Why

Evaluated work is judged under detection heuristics and integrity rules, not only technical merit:
out-of-policy help can zero a submission that was technically perfect, a far worse outcome than
slower or more limited assistance. Stating the boundary explicitly also protects the requester's
standing and keeps the agent's contribution auditable - setup and verification help are normally
permitted, authorship normally is not.

## Exceptions and boundaries

Practice exercises, personal projects, and explicitly ungraded exploration carry no such constraint.
If the policy permits a defined amount of AI assistance with disclosure, help within that limit and
note the disclosure requirement in the deliverable. Refusing legitimate support such as running the
environment is over-correction; deciding on the requester's behalf to hide assistance is out of
bounds regardless of the task.

## Example

A graded exercise is submitted as a report plus a source archive, and the integrity policy warns
that obviously machine-written work can lose full marks. The agent installs and verifies the
environment, checks the candidate's numbers against actual runs, and explains the method; the
written solution stays with the candidate, and a provided plotting utility is reused instead of a
custom rewrite because the rubric requires it. The constraint is restated at each later deliverable,
which inherits the same policy.
