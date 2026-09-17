---
{"id":"research-ops.testing.assert-invariants-in-verification","title":"Land Invariants as Assertions and Verify at the Consumer","stage":"testing","tech_stack":["research","ml","agent-ops"],"applies_when":"a component promises isolation, read-only access, or a quota, or a fix is being accepted on the strength of attribute values, log lines, or green unit tests","severity":"warn"}
---

## When to apply

Apply whenever a guarantee of the form "X must not affect Y" is written down: evaluation must not mutate
live state, inspection is read-only, quotas must hold, configuration must reach the objects that use it.

## Guidance

Turn every such promise into an assertion that fails first: a docstring saying "this path does not mutate
the environment" is a wish, while a failing test is a constraint.

Design acceptance around the threat model, not around history. Enumerate the call sites that rely on the
isolation guarantee and check each trigger surface; a defense built for the previous incident usually does
not cover the next one. Ask "which combinations can reach this path", not "did the last scenario pass".

Keep at least one test on the real engine and the real construction path. Stub and table-driven tests
verify plumbing, not physics: they stay green while a real run crashes or mutates live state. New
entry-point code must execute on the real wrapper stack; source-level wiring checks do not count.

Accept fixes at the consumer, not at the attribute. Setting a field does not update schedules, optimizer
state, or buffers built earlier from the old value; assert the behavior that consumes the value (the step
that uses the rate, the tensor carrying the discount) and re-verify dependents after late injection.

Close anomalous signals with arithmetic before attributing them to capability. A plausible denominator does
not validate the process quantities: compute the physical minimum (turns needed to lose a given amount of
health, steps before termination) before concluding "that is normal for a weak baseline".

Audit upstream trust boundaries: find the idioms that silently break an assumption you depend on (copying
that hides closures, handles that alias shared mutable state), write the dependency down, and test it.

Validate your self-checking tools. A scanner deciding "copy or original" can have its predicate inverted by
the key/value direction of the structure it inspects; cross-check structural scans with behavioral
assertions and suspect the predicate first when they disagree.

Route exceptions through a structured channel. A broad handler that converts a crash into a normal outcome
(a loss, a negative reward, a terminated flag) yields reports of "zero alarms" while monitoring is blind:
absence of log text is not absence of exceptions, and every catch that continues down a normal path is
where monitoring stops. For quotas, make the reserved unit equal the consumed unit, and keep downgraded
artifacts out of main statistics.

## Why

These gaps are complementary: each layer looks legitimate on its own, so a defect reaches production only
by surviving all of them at once. The invariant is never asserted, the defense covers the previous trigger
surface, the stub cannot see the real path, and the swallowed exception erases the evidence.

## Exceptions and boundaries

Pure logic does not need an end-to-end engine test, but any claim about live-state isolation or real-engine
behavior needs one real-path test. Arithmetic checks are sanity gates, not statistical validation.

## Example

A teacher-evaluation path documented as read-only writes through a captured reference into a live episode,
and the invariant becomes a failing test on its first run. Later, a fix prints "applied" and passes its
guard tests while the schedule and optimizer keep old values, because only the model attribute was set; the
accepted verification asserts the learning rate the next optimization step actually uses.
