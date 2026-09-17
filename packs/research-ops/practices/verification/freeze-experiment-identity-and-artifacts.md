---
{"id":"research-ops.verification.freeze-experiment-identity-and-artifacts","title":"Freeze Experiment Identity and Artifacts Before Running","stage":"verification","tech_stack":["research","ml","agent-ops"],"applies_when":"an experiment is about to launch or be reported and its environment, seeds, checkpoints, state sets, or code hashes are not yet frozen into one immutable auditable bundle","severity":"warn"}
---

## When to apply

Apply before any run whose numbers will be published or drive a decision, and whenever the same experiment
is resumed, re-run, or compared across time. An incomplete identity chain is a blocker, not a detail to
backfill after the run.

## Guidance

Define the experiment as an identity chain: environment era, seed set, model artifacts, state or probe
sets, algorithm interface and implementation, harness code, serialization version. Hash each component and
attach a manifest listing the hashed objects - paths, fields, ordering - so the hash boundary can be
reconstructed. A bare hash without a manifest is pseudo-freezing and makes the run invalid.

Hash whole files with sha256, not parameter subsets, and bind each artifact id to its hash immutably so a
content change becomes a new identity. Never point a registry at mutable names such as latest, best, or
final: the spelling survives while the bytes change. Keep the semantic identity (policy, observation
format, serialization version) separate from the file hash, because either can move independently.

Do not let a name pre-judge a result. Carry an explicit unverified status until the equivalence gate has
run, and register the new era, branch, or status label only if the gate fails. Naming first and letting
the experiment decide what the name meant is a circular definition.

Freeze as one bundle in dependency order: schemas, then generated entities, then hashes, then the lock,
then the run. Generating seeds, states, or artifacts mid-flight and backfilling hashes invalidates every
gate that ran earlier. If an allocation misses required coverage, regenerate the whole allocation under a
new manifest id; never patch in the missing item.

Make the identity anchor deterministic: canonical, order-independent serialization that does not depend on
process address layout or hash seeds. Verify that replay reproduces the frozen hash, and treat a failed
replay of the pre-fix stack as a finding about the anchor rather than as noise. Prove cross-process
determinism with a large replay test, not a small offline sample.

Freeze selection independently of outcomes: state sets, probe sets, and strata are experiment assets and
must not be sampled from observed results. If the candidate distribution is induced by one policy, report
blind selection as execution-regression coverage, not as representative sampling.

## Why

Numbers are interpretable only when the thing measured is the thing you believe you measured. Unfrozen
identity lets a result drift silently - a resumed run inherits different seeds, a checkpoint is overwritten
under the same name, a state set is quietly trimmed - and no later analysis can tell. Freezing costs hours;
not freezing costs the ability to answer the question at all.

## Exceptions and boundaries

Exploratory smoke runs need not carry the full chain, but they must be marked so they cannot be promoted
to evidence later. A superseded artifact is retired by writing a new id, never by editing the old record;
the diagnostic value of the old hash is exactly why it was kept. Freezing establishes identity, not
quality: a frozen artifact is not thereby endorsed.

## Example

A registry entry pairs a role name with the sha256 of the entire file - an internal smoke-test filename is
still the frozen artifact, because the registry defines identity, not the name - plus the policy identity
triple. A candidate pointer named "best" is rejected because its bytes can change under the same name, and
a re-exported checkpoint with identical weights but different bytes is registered as a new artifact id.
