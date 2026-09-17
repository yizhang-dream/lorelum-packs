---
{"id":"toolchain-deploy-ops.implementation.check-contract-seams-between-agents","title":"Check Cross-Agent Contract Seams at Integration","stage":"implementation","tech_stack":["agent-ops","toolchain"],"applies_when":"two or more agents implement separate files against a shared interface - data shapes, DOM hooks, global symbols, or load order - and their work is about to be integrated","severity":"warn"}
---

## When to apply

Apply whenever work is split across agents that own separate files but share one interface: a producer
emitting a data structure and a consumer rendering it, a component writing class names and a stylesheet
that must style them, or files whose load order decides what exists at initialization time. Apply at
the integration step, before the feature is declared done.

## Guidance

Enumerate the seams explicitly before integration: field and key names on both sides, DOM or
class-name hooks, global symbols and registration keys, initialization order, and default values for
optional shapes. Put the contract verbatim in each dispatch instruction instead of assuming "the
obvious name" is shared.

At integration, grep each shared identifier on both sides and open the files that must agree. When a
mismatch appears, accept both spellings at the consumer for immediate repair, then fix the canonical
name at the producer in the same pass - never leave two names in flight. Treat a missing style rule for
an emitted class, or a container assumed but never created, as the same class of defect as a wrong
field name.

## Why

Parallel agents cannot see each other's in-progress reasoning, so each one fills an underspecified
interface with its own plausible choice. The resulting failures are usually silent rather than loud: an
unstyled element still renders, a read of an undefined field yields an empty label, a renderer that
never receives its data draws nothing. None of these trip a type checker or a build, so they surface
only if integration checks the seam deliberately.

## Exceptions and boundaries

Do not accumulate permanent alias layers - each tolerated duplicate name is future confusion. A seam
check is unnecessary when a single agent owns every file the change touches. Defensive reads at the
consumer are a bridge, not a contract: the producer's canonical name is the contract, and the bridge
should be removed once the producer is fixed.

## Example

A renderer is dispatched to consume a graph structure while the data generator emits the same
structure under a different key. The renderer reads its own name for edges and silently draws nodes
with no links; separately, the class names it emits for labels have no stylesheet rules anywhere. Both
defects survive every unit check and appear only at integration. Fixing them means accepting both keys
at the consumer in the short term, renaming the producer in the same pass, and grepping every emitted
class name to confirm a style exists.
