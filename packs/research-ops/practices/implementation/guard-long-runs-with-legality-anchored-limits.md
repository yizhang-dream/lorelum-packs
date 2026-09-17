---
{"id":"research-ops.implementation.guard-long-runs-with-legality-anchored-limits","title":"Guard Long Runs With Legality-Anchored Limits","stage":"implementation","tech_stack":["ml","research"],"applies_when":"a long simulation or training run can exhaust memory or hang, while the domain has legitimate behavior that looks unbounded, so a naive cap would forbid legal cases","severity":"warn"}
---

## When to apply

Apply when a training or simulation job can grow without bound - pathological episodes, self-loops,
runaway workers - and a naive limit would also forbid legitimate behavior. Also apply when a watchdog
already exists but the machine still dies or the job still hangs.

## Guidance

Separate two responsibilities instead of searching for one magic number:

- In-domain guard (protects correctness): cap activity at a level anchored to the measured legal
  distribution - observed maximum times a safety factor - never a round guess. Convert the trip into a
  semantics-preserving action: for example, force a turn boundary so the entity resumes on a normal
  path and keeps its capability on the next turn.
- Process-level isolation (protects the machine): give each worker a hard memory limit enforced by the
  kernel (cgroup or rlimit) so a runaway kills one process instead of triggering a machine-wide OOM.
- Make trips loud and traceable: a trip means a bug ticket. Discard the offending episode, record its
  seed or state fingerprint in a differential audit ledger for replay, and never emit it as data.
- Make the job resumable so a kill costs one shard: sharded collection plus training checkpoints with
  resume.
- Audit where the watchdog cannot look. A timeout checked inside one call path never fires for a loop
  that does not call it: summon chains, refill loops, and trigger cascades can spin while the guard
  reports zero trips and the parent process sits blocked for minutes. Extend checkpoints to every hot
  loop, not just the obvious entry.
- Recognize the failure signature: a worker memory plateau reached with zero trips plus a parent
  blocked on a long receive means an unwatched loop.
- Check for amplifier costs before blaming the workload: process-fork inheritance of a full heap, and
  closures that pickle entire resource pools, multiply baseline memory across workers.

## Why

Legitimate unbounded behavior and runaway bugs look identical from outside, so a single hard cap either
forbids legal cases or fails to protect the machine. Anchoring in-domain limits to measured data keeps
legal behavior available, while process-level isolation guarantees that any remaining bug costs one
worker and one ticket instead of an unusable machine and an interrupted run.

## Exceptions and boundaries

Process-level time or memory limits must stay outside the data-producing path; a limit that changes
teacher or reference outputs breaks reproducibility and is not a valid guard. If trips are frequent,
fix the underlying bug rather than raising limits. Never raise an in-domain limit without a fresh
measured distribution justifying the new value, and re-measure the legal distribution whenever the
domain surface expands.

## Example

One worker grew from about 2 GB to nearly 7 GB and was killed mid-run. Forensics showed that of more
than a thousand historical worker memory samples, none had exceeded 2 GB, and the guard reported zero
trips because its checkpoint sat only on the card-play path - the looping code never reached it. The
fix was two-sided: extend trip checkpoints into the non-card-play loops, and implement the per-worker
hard memory cap that had been designed but never built.
