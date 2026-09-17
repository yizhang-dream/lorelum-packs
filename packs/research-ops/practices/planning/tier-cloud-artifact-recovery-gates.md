---
{"id":"research-ops.planning.tier-cloud-artifact-recovery-gates","title":"Tier Cloud Artifact Recovery and Block Analysis Until Verified","stage":"planning","tech_stack":["ml","research"],"applies_when":"a long or expensive job runs on rented or remote compute whose storage is ephemeral and whose shutdown may not trigger artifact upload, or artifacts from such a run are about to be analysed","severity":"warn"}
---

## When to apply

Apply at planning time, before the job is launched, and again at retrieval time. Any compute whose
lifecycle is controlled by a provider - spot instances, rented GPUs, batch platforms - qualifies, and
so does any local run whose only copy of the evidence would die with the machine.

## Guidance

Treat the remote filesystem as a cache, never as the source of truth. Plan the artifact path in
advance: an atomically written local bundle (write to a temporary path, flush, fsync, rename) plus
evidence embedded in the run's own log stream (receipts, full training logs, audit output) plus a hash
manifest. The shutdown moment can be the only retrieval window, so the bundle must be complete before
the process exits; an audit or finalisation script is part of the transaction's critical path, not an
optional sanity check - if it fails, the receipts stay remote and the evidence does not exist.
Self-test that script on known-bad inputs, including type contracts between the training stack and the
analysis stack.

Freeze the recovery order and do not reorder it to peek at results: enumerate artifacts, download all
of them, unpack bundle and receipts and manifest, re-verify every hash, and only then allow scientific
readout. Assign a recovery tier to what came back - fully auditable, scientifically recoverable, or
descriptive salvage only - and never present salvage as a complete auditable experiment. Keep
result-recoverable distinct from audit-recoverable. On failure, rerun only through a pre-authorised
path and never merge lineages: data from a pilot and its canonical rerun are different populations.
While a run is in flight, do not expand the matrix or add datapoints; the priority is preserving the
existing evidence chain.

Schedule by measured bottleneck, not by convenience: escalate to larger or rented compute only when the
measured bottleneck is on the accelerator, and keep deterministic reproduction and hash-comparison
jobs on their original machine, because byte-identity across machines is not guaranteed.

## Why

On rented compute the dominant risk is not capacity but artifact survivability: a platform can stop a
job with no error in the logs, and a filesystem that vanishes takes the only copy of the evidence. The
ordered gate exists because analysis before verification is a ratchet: once results are visible, the
tolerance for gaps in the evidence quietly increases. Tiers keep the claim's strength tied to what was
actually recovered.

## Exceptions and boundaries

If artifacts are already local, or the store is durable and checksummed end to end, the gate shortens
but the order stays: verify, then read. A recovery tier below fully auditable is a statement about the
evidence, not a verdict on the science, and should be reported as such. Pre-authorised reruns after a
genuine failure are legitimate; merging a failed lineage's numbers into the new one never is.

## Example

A remote training job stops mid-run with no errors in its log, and the archiving action recorded just
before the stop identifies it as a provider-side termination rather than node maintenance. The operator
downloads the full artifact list, re-verifies the hashes, classifies the bundle as scientifically
recoverable rather than fully auditable, and keeps the analysis blocked until that verification
completes; the earlier pilot run is never merged with the canonical rerun.
