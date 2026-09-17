---
{"id":"toolchain-deploy-ops.implementation.anchor-follow-cams-to-stable-ids","title":"Anchor Follow Cameras to Stable Entity IDs","stage":"implementation","tech_stack":["toolchain","windows"],"applies_when":"a viewer or camera follows one entity in a simulation whose entity storage is compacted or reordered between frames, or a spawn-based multiprocessing pool must be launched from a script","severity":"warn"}
---

## When to apply

Apply when writing follow, lock-on, or selection behavior over a simulation snapshot, and when
parallelizing a workload on a platform whose process start method is spawn.

## Guidance

Give every entity a stable identifier at creation and carry that identifier in the frame contract.
Anchor the camera to the identifier and resolve it to a position each frame. Never anchor to an array
index: compaction (removing dead entities) shifts every later element, so an index that pointed at the
tracked entity silently points at a different one, and the symptom reads as a rendering or physics bug
rather than an identity bug. When the storage buffer is packed, keep an identity map from the stable id
to the current row and rebuild it after every compaction.

For parallelism on spawn-based platforms, start process pools from an importable module and a
command-line entry point. Pool workers re-execute the interpreter and import the entry module; when
that entry is a heredoc piped into the interpreter on stdin, there is no importable name to re-execute
and the pool fails or, worse, runs nothing while reporting success. Refactor the work into a module
subcommand first, then start the pool from that.

## Why

Both are identity failures across a boundary. An array index is a position, not an identity - any
compaction invalidates it, usually without an error, because the stale index remains in range. A stdin
script is anonymous code with no importable name, so a spawn-based start method cannot reconstruct it in
its workers. In both cases the code behaves correctly in the single-entity, single-process case that is
tested first, and only misbehaves under the dynamics the feature exists for.

## Exceptions and boundaries

Index anchoring is acceptable while the container is append-only and never reordered - state that
invariant explicitly if it holds today. Platforms that use fork tolerate stdin entry points, but the
module entry point is the portable form; write that anyway. The identity map must be rebuilt from the
same snapshot that produced the positions, or the camera lags a frame across a compaction.

## Example

A colony viewer followed one ant by its slot in the packed arrays. After deaths compacted the arrays,
the camera locked onto whichever individual had slid into that slot and the view jumped to an unrelated
part of the map; it was reported as a rendering glitch and diagnosed as an identity bug only after a
stable per-individual id was substituted. In the same project, launching the analysis pool from a stdin
heredoc failed under the spawn start method - moving the work into a module subcommand and invoking
that entry point made the pool start reliably.
