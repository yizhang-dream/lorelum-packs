---
{"id":"app-delivery-ops.testing.keep-living-regressions-on-critical-paths","title":"Keep Living Regressions on Critical Paths","stage":"testing","tech_stack":["web","agent-ops"],"applies_when":"a fix or rework lands on a critical end-to-end path whose unit tests and offline checks are green, and the path has not been exercised against the real running system before","severity":"warn"}
---

## When to apply

Apply after any repair, migration, or framework swap on a path that a user or downstream service
depends on end to end - ingestion, delivery, sync, notification - especially when the breakage was
found by inspection rather than by a failing test. Also apply before declaring such a path healthy.

## Guidance

Define the shortest possible live exercise of the critical path and keep it as a standing regression:
one real request, message, upload, or command through the production-shaped path, with a specific
observable success signal. Run it before and after changes that touch the path.

- Prefer the live path over replay scripts, mocks, and offline harnesses; those prove the parts, not
  the connection between them.
- Exercise through the same entry point a real caller uses, not a shortcut that bypasses transport,
  command matching, permissions, or queueing.
- Assert on the response or side effect that a caller can see. A green log line, a started process,
  or an opened port is not a success signal.
- Keep the recipe short enough to run routinely - a single command or a short checklist - and record
  what it proves next to it.
- When the path cannot be exercised live, say so explicitly in the report and name the substitute
  check; do not present unit tests as end-to-end evidence.

## Why

Offline tests are written against the code that exists when they are written. A path can be dead in
production while every unit test passes: a router rule never matches, a synchronous call crashes
inside an async handler, a migration predates the feature layer that was later added. Those are
exactly the failures that offline suites do not see, and they can persist for weeks because nothing
pings the real path. A living regression is the only check that keeps the wiring honest.

## Exceptions and boundaries

Do not point live regression scripts at production data or destructive operations; use a test
channel or a reversible action where possible. A live check on a shared service should be
rate-conscious and identifiable. For irreversibly destructive paths, a staged dry-run plus one
supervised real execution at release time is the acceptable substitute.

## Example

A full-repository review finds that an ingestion handler stopped receiving messages several
releases earlier because a matcher rule was never attached, while the test suite stayed green
throughout. The fix ships with a one-command live exercise: send a real command through the normal
entry point and check that a reply arrives. That exercise becomes the standing gate run before each
release, because it is the only check that would have caught the original breakage.
