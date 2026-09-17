---
{"id":"research-ops.implementation.kill-homogeneous-compute-on-clear-signal","title":"Kill Homogeneous Compute When the Signal Is Already Clear","stage":"implementation","tech_stack":["research","ml"],"applies_when":"a multi-seed or full-budget experiment batch is still running while an intermediate checkpoint already shows a decisive pattern (early peak then collapse, a recipe bug, a failure mode) and the remaining runs would only repeat a settled outcome","severity":"warn"}
---

## When to apply

Apply while an experiment batch is in flight, not only in retrospect: whenever the intermediate evidence
already answers the question the remaining runs were budgeted to answer. Typical triggers are an early
peak followed by degradation, an obviously wrong recipe or configuration, or an error form (overspeed,
falling) that is already familiar and decisive.

## Guidance

Pre-registration governs what counts as evidence; it does not require spending the remaining compute on
a question that is already settled. Keep the judgement criteria, metric identity, and readout rules
frozen, and treat the execution budget and the follow-up iterations as adjustable. On a clear signal,
do all four of the following rather than one of them:

1. Stop the remaining homogeneous runs.
2. Salvage existing artifacts - frequent checkpoints exist precisely so a partial run can still fill
   key gaps.
3. Implement the fix immediately and launch the next iteration.
4. Record in the ledger where the protocol was amended mid-flight and why.

Step 4 is what keeps this compatible with pre-registration discipline: the judgement standard did not
move, only the budget for producing more instances of the same result.

## Why

Compute is the binding constraint in small-lab research. A pre-registration recipe that insists on
running every seed to full budget after the outcome is visible converts scarce GPU hours into copies
of information already obtained, and it delays the fix that the observed failure is asking for. Fast
iteration also concentrates learning per unit of wall clock: each observed defect becomes an immediate
design change instead of a note for a later planning cycle.

## Exceptions and boundaries

An ambiguous or noisy signal is not a clear one; if the intermediate pattern could plausibly reverse,
let the batch finish. Protect runs that carry a second duty - reproducibility or hash-baseline runs
that must complete in their original environment. Never retroactively reinterpret the criteria to make
the early stop look pre-planned, and do not use "we will fix it later" as a reason to skip step 1.
Stopping is a budget decision, not a licence to change what counts as success.

## Example

A training run peaks at an early checkpoint and degrades afterwards, in a batch where later seeds would
consume many more hours. The operator kills the remaining seeds, salvages the checkpoint series to fill
the coverage gaps, converts a planned ablation into the fix arm, and launches it the same day, then
writes in the log that the batch was shortened mid-protocol because the peak-and-collapse form was
already decisive.
