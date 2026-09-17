---
{"id":"toolchain-deploy-ops.implementation.prefer-ecosystem-orchestration-over-scripts","title":"Prefer Ecosystem Orchestration Libraries Over Hand-Rolled Scripts","stage":"implementation","tech_stack":["toolchain"],"applies_when":"a parameter sweep, analysis pass, or visualization step for a simulation or scientific workload is about to be hand-written as batch shell or Python code, or the workload is about to be moved to a GPU backend","severity":"warn"}
---

## When to apply

Apply when planning batch runs, trajectory analysis, or rendering for an engine whose ecosystem ships
dedicated tooling, and before switching any workload from CPU to GPU.

## Guidance

Measure before changing hardware or writing infrastructure. Profile a representative run and attribute
wall time by phase. When the engine itself is a small fraction of the total while the surrounding
bookkeeping dominates, a GPU backend is a negative optimization: every step then pays a device-to-host
synchronization to read state back, which the CPU path got for free. The switch is one line, but the cost
is paid each step, so gate it on problem size - the engine's native regime - not on the device being
present.

For orchestration, use the ecosystem's components instead of batch scripts: its parameter-and-workflow
framework for sweeps, library implementations for clustering, correlation, or density analysis, and the
community trajectory format for output. Every hand-rolled replacement is code you now own, with no
upstream tests behind it. Treat the distribution channel as part of the choice - a package that ships
only through an environment manager is not a drop-in pip dependency, and mixing channels is the usual
cause of import failures. Visualization follows the same rule: a headless engine has no viewport, so the
standard path is to write a trajectory during the run and render afterwards with the ecosystem's viewers
or batch renderer.

## Why

Reimplementing analysis looks cheap and becomes a permanent correctness and maintenance liability, while
the ecosystem implementations are the ones the field validates against. GPU-first thinking is the same
trap on the compute axis: profiling commonly shows the engine is not the bottleneck at small scale, so
the acceleration target must be chosen from data rather than from the specification sheet.

## Exceptions and boundaries

Ecosystem tools can be the wrong fit - for example, an engine that imposes a global reaction table when
the project's own invariants forbid one, or a plugin line that needs a compiler toolchain for a one-off
need. Justify an exception with a written comparison, not with convenience. Hand-written orchestration
is still reasonable for a handful of runs where the framework's setup cost would dominate.

## Example

A soft-matter simulation project planned to accelerate with the GPU until a profile attributed roughly
5 percent of wall time to the engine and the large majority to the project's own accounting layer - at
toy particle counts the GPU would have been slower. The same project adopted the ecosystem's sweep
framework for its parameter searches, its analysis library for cluster and density statistics, and its
trajectory format plus an external viewer for all visual output, because the engine is headless and has
no UI of its own.
