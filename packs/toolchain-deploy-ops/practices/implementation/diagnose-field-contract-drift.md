---
{"id":"toolchain-deploy-ops.implementation.diagnose-field-contract-drift","title":"Diagnose Silent Feature Loss as Field Contract Drift","stage":"implementation","tech_stack":["deployment","toolchain"],"applies_when":"a feature that used to have an effect stops producing one after a merge or refactor, the code path looks intact, and nothing anywhere in the request errors or warns","severity":"warn"}
---

## When to apply

Apply when a capability silently degrades after an integration point changes - an attachment is never
injected, a preprocessing step never runs, a record never reaches the model - while the surrounding
pipeline still reports success. The tell is a feature failing with zero errors and zero warnings.

## Guidance

Instrument the whole chain with a real payload before touching code: exercise producer and consumer
end to end and confirm which layer stops. Then compare the condition the consumer gates on with the
field the producer now sends. A merge often moves a decision key from one field to another (a
capability name becomes a workspace mode) while the gate keeps reading the old field, so it never
matches and the branch is dead code that still looks correct in review.

Read the persisted request snapshot as evidence, but know its limits: recorders store only the fields
they were written for, so a field absent from the snapshot may still have been sent. The
authoritative evidence for "did the content reach the model" is the consumer's own reasoning trace,
not the absence of an error.

Fix the gate to read the field the current contract carries, and accept both request shapes while
legacy callers exist. Do not stop at the first drift: validators on the same value drift too. A
producer that starts minting prefixed identifiers will be rejected by a strict format validator
downstream, which turns the value into an empty string and reproduces the same silent loss. Grep
every validator of that field and widen them together.

Also watch transient lifecycle states: an entity still being processed is often dropped by the client
before the request is built, which looks identical to a contract bug from the consumer's side.

## Why

Two teams can both be correct about their own layer while the seam between them is broken. Nothing
crashes because the gated branch simply never executes, and the component that still activates by the
newer key keeps claiming the work is done, so every observable signal reads healthy. Only a
layer-by-layer trace with real payloads, plus the consumer-side trace, reveals the empty promise.

## Exceptions and boundaries

Not every silent loss is contract drift - a provider outage, an authorization failure, or a client
that never attached the asset all produce the same symptom. Confirm the producer actually sent the
value before rewriting the consumer. When several consumers duplicate a validation rule, fixing one
is not a fix; record the remaining sites as known debt with their impact.

## Example

An image-reading assistant stops attaching page images after a merge. The extractor, the media
endpoint and the frontend renderer all test clean; the gate that attaches images still keys on a
capability field, while the client now sends a generic capability plus a workspace mode, so the gate
never fires. The persisted trace shows the model reasoning "no image content provided in this turn"
and no warning events - nothing was attached, rather than stripped afterwards. Repointing the gate at
the workspace mode restores injection; a second bug surfaces the same day when re-imported documents
get prefixed identifiers that a hex-only validator rejects. Both are one class: the producer moved,
the consumer did not.
