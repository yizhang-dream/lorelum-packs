---
{"id":"toolchain-deploy-ops.implementation.serialize-heavy-rebuilds","title":"Serialize Heavy Rebuilds Behind One Queue and Stagger Them","stage":"implementation","tech_stack":["deployment","toolchain"],"applies_when":"two or more heavy jobs - index rebuilds, production builds, bulk parsing, model-backed ingest - are about to run at the same time on one machine, or a rebuild has just failed with a resource-exhaustion error or a stale lock","severity":"warn"}
---

## When to apply

Apply before launching any job whose cost is measured in tens of minutes and which uses a shared,
non-partitionable resource: an embedding service, a build tool, the accelerator, pagefile or commit
charge, or a build directory guarded by a lock file.

## Guidance

Put every job in that class through one serializing owner:

1. Cap concurrency at one. A global queue with a single worker, or a single-flight lock around the
   rebuild entry point, is enough; ad-hoc parallel invocations are the failure mode. If jobs can be
   started from different surfaces - CLI, web UI, agent - they must all enter through the same
   guard.
2. Preflight the shared resource. Before starting, check free memory or commit headroom and whether
   another heavy producer (a model server, a renderer, a bulk transcription job) is active; wait or
   stagger if it is.
3. Expect and handle half-finished runs. A rebuild killed mid-flight may leave a lock or an
   incomplete state marker, which makes the next attempt rebuild from scratch or refuse to start.
   Re-running the idempotent launcher is usually the fix; do not hand-delete locks first.
4. Make the queue observable. Expose queued, running, and finished states so a second operator can
   see that a rebuild is in flight instead of starting another one.

## Why

These jobs contend for resources that cannot be divided: parallel index rebuilds can exhaust the
socket buffer of the embedding service, a production build beside a bulk OCR job makes both slower,
and their combined commit charge can push the machine into paging or process death. The failure is
rarely a clean error - it is an OS-level resource exhaustion, a silent exit, or a stale lock that
turns the next run into duplicated work. Serial execution costs wall-clock time; parallel execution
costs the same wall clock plus an unpredictable failure mode.

## Exceptions and boundaries

Light, I/O-bound derivations - per-item API calls, sentence translation with a small worker pool -
are parallelizable and should stay parallel. Serialization is per machine and per shared resource,
not per job type: two genuinely independent machines need no shared queue. A queue does not fix a
job that is simply too large for the machine; enforce a headroom threshold in the preflight and
refuse to start instead of dying halfway.

## Example

Two index rebuilds launched together both died with a socket-buffer error from the embedding
service; running them one after another completed normally. Later, a front-end production build
started beside a bulk OCR job slowed both, and the build died mid-run leaving its lock in place - a
launcher restart then rebuilt fully, hit an "another build is running" error, and self-healed on a
retry. Both symptoms disappear once the heavy class is queued and preflighted.
