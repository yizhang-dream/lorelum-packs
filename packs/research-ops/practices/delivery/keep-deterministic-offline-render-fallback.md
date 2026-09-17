---
{"id":"research-ops.delivery.keep-deterministic-offline-render-fallback","title":"Keep a Deterministic Offline Render Fallback","stage":"delivery","tech_stack":["simulation","agent-ops"],"applies_when":"a deliverable depends on live GUI capture or interactive tooling that may be unavailable, unstable, or unrepeatable at submission time","severity":"warn"}
---

## When to apply

Apply when a report, demo, or figure must be produced from a long simulation or experiment and
the primary production path is interactive: screen recording, a live GUI, a manual session.
Also apply before promising that a key artifact can be regenerated on demand.

## Guidance

Maintain a second, fully offline path that renders the same content from stored run data with
no GUI, no window manager, and no interactive input, and make it byte-reproducible: fixed
frame schedule, fixed encoding parameters, and a hash recorded alongside the output. Two runs
of the offline path must produce identical bytes, so a reviewer can re-run it and compare
hashes instead of trusting a description of the render.

For long runs, drive the pipeline as a chain of bounded segments rather than one process that
must survive for hours. Each segment checkpoints its result, and the chain script appends a
one-line machine-readable summary per segment to a shared log. That log then carries the
run's story - segment boundaries, key metrics, and the resume point after a crash - without
re-reading raw output.

## Why

GUI-dependent capture fails for reasons unrelated to the science: focus stealing, environment
differences, codec and driver variance. When the deadline arrives, a fallback that works
offline from persisted data is the difference between shipping and re-running. Byte-level
reproducibility turns "the render is correct" from an assertion into a checkable fact, and
segmented execution bounds the blast radius of any single failure while leaving an auditable
trail.

## Exceptions and boundaries

A deterministic offline render cannot show real-time interaction or timing-dependent UI
behavior; keep the interactive capture as an optional extra when it succeeds, but never let it
be the only source of the deliverable. Byte-identical output also assumes fixed library and
font versions - record them next to the hash, or the guarantee silently expires.

## Example

A simulation deliverable shipped as an offline render: a fixed set of frames generated
directly from stored run files, reported with the output hash. Live capture of the same run
stayed the optional version, so when the GUI path later failed for environment reasons, the
shipped artifact was already valid and reproducible.
