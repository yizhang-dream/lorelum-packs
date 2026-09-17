---
{"id":"app-delivery-ops.implementation.match-lock-scope-to-process-model","title":"Match Lock Scope to the Deployment Process Model","stage":"implementation","tech_stack":["web"],"applies_when":"adding mutual exclusion around file-based or document-based read-modify-write storage, and choosing between an in-process lock and a cross-process one","severity":"warn"}
---

## When to apply

Apply when several code paths read a stored document, modify it in memory, and write it back
(sessions, one-time codes, rate limits, per-user data buckets). Also apply when reviewing whether
existing atomic writes are enough protection.

## Guidance

Separate the two protections: an atomic write (write to a temp file, then rename) prevents torn or
half-written files, while a lock prevents lost updates from interleaved read-modify-write cycles.
Both are needed; atomic writes alone still lose data, because two writers can read the same version
and the later rename erases the earlier writer's unique additions.

Then match the lock's scope to the process model you actually deploy:

- A single long-running process can use an in-process promise chain keyed per file or per resource.
- Anything that can run more than one process (cluster mode, multiple hosts, serverless, or an
  orchestrator that restarts and overlaps instances) needs an OS-level file lock, a database
  transaction, or a single-writer service. An in-process lock across processes is no lock at all.

Cover every shared writer with one named primitive rather than several ad-hoc mutexes, prefer
resource-level granularity (per file, per user bucket), and write the single-process constraint in
both the code comment and the deployment documentation. When contention or write complexity grows
past what the primitive can express, move storage to a transactional database instead of extending
the lock.

## Why

The dangerous version of this bug is invisible in development: with one process the ad-hoc lock
appears correct and tests pass, and the failure appears only after scaling out. Naming the
constraint at the primitive and in the deployment docs is what makes the future migration a
decision instead of an incident.

## Exceptions and boundaries

Read-only paths need no lock. A lock does not make an unsafe merge strategy safe: last-write-wins
convergence is a separate product decision and should not be re-invented inside the lock. Locking
raw uploads of the whole document is fine at small scale but does not scale to large documents.

## Example

A server persisted sessions, one-time login codes, rate-limit counters, and per-user data files,
each with its own atomic write, yet concurrent requests still double-consumed codes and dropped
entries from a bucket. A single per-resource promise-chain lock wrapped all four resources, with a
comment marking it single-process-only, and the concurrency gate of one hundred parallel writes
then passed with zero loss.
