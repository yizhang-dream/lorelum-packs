---
{"id":"research-ops.planning.pin-shared-conventions-in-dispatches","title":"Pin Shared Conventions in Dispatch Instructions","stage":"planning","tech_stack":["agent-ops","research"],"applies_when":"several agents or workers will modify similar objects such as assets, components, schemas, or data files in parallel, and each worker could otherwise choose its own values for shared dimensions, constants, units, or naming","severity":"warn"}
---

## When to apply

Apply before dispatching a parallel wave whose tasks touch sibling objects that must stay mutually
consistent: repeated components, grid or unit conventions, shared constants, field names, or
threshold values. The risk is highest when each worker sees only its own slice of the repository.

## Guidance

Two steps, both done by the dispatcher before any worker starts:

1. Reconcile the object list against the repository. Grep the whole tree for the object classes
   being changed and build the complete list of affected files. Compare it with the scopes assigned
   to each worker: every file must be claimed exactly once. A file that appears in no scope is the
   most common omission and the hardest to notice after the wave, because no worker will report it.
2. Pin the shared values verbatim in every instruction. Quote the exact constants, units,
   quantization or grid rules, naming patterns, and which fields are frozen, as one copy-pasteable
   block reused across all dispatches. Do not rely on workers inferring conventions from examples,
   nearby files, or a design document they may not open.

Name the files each worker owns in its instruction. If two workers must touch the same file,
serialize them or split the file into disjoint regions with an explicit order.

After the wave, run an independent pass that checks two things: no file outside the reconciled list
changed, and no listed file was missed. Treat a missed file as a dispatch defect, not a worker
defect.

## Why

Independent workers each make locally reasonable choices. Without a pinned value they drift: sibling
objects end up with different dimensions, field names, or thresholds, and the merge is inconsistent
in a way no single worker could have detected. The complete repository list closes the other gap,
unowned files, which survive review because everyone assumes someone else handled them.

## Exceptions and boundaries

Files deliberately deferred to a later wave should be marked as such in the same list so they are
not mistaken for omissions. A convention that genuinely differs per object should be stated as an
explicit per-file value rather than as a rule with exceptions. When the shared value is not yet
decided, decide it before dispatch - a wave that has to reconcile conventions afterwards costs more
than the decision.

## Example

A wave re-quantizes several dozen similar visual components with measurable sizes and placements.
The dispatcher first greps for the component classes and produces the full file list, then writes
the canonical size and placement values plus the owned file list into each worker instruction. A
previous wave had failed review because two component files belonged to no worker's scope; the
list-first step catches them before dispatch instead of in review.
