---
{"id":"app-delivery-ops.implementation.expect-startup-init-order-hazards","title":"Expect Startup Init Order Hazards from Immediate Dispatchers","stage":"implementation","tech_stack":["mobile"],"applies_when":"a screen-scoped state holder launches initialization from its constructor on an immediate or undispatched main-thread scheduler and the app crashes on open while unit tests and pure function tests stay green","severity":"warn"}
---

## When to apply

Apply when an initializer block starts work on the main thread with an immediate dispatcher, when a
constructor-time coroutine touches state declared elsewhere in the class, or when a release crashes at
startup on a path that has no test coverage. Also apply when reviewing any recurring "block that runs at
construction" pattern.

## Guidance

An immediate main-thread dispatcher does not schedule the block - it inlines the body synchronously up to
the first suspension point. Two consequences drive the rules:

- Make the first statement of the init block a real suspension point (a collected stream, a suspending
  read). That yields execution back before any member access and turns the remainder into normal
  post-construction work.
- Declare every property the init block touches *before* the init block. Reading a property declared later
  happens before its initializer has run.
- Lock the constraint where the next editor will see it: a comment at the class top stating the ordering
  requirement, and a review check for any new statement placed ahead of the suspension point.

Move initialization out of the constructor when the class is not required to self-start; explicit start
calls from the lifecycle owner make ordering visible instead of relying on declaration order.

## Why

Coroutine builders with an immediate dispatcher are documented as "may execute in the caller's thread until
the first suspension", and the caller here is the constructor. A synchronous first line that reads a
not-yet-initialized property therefore dereferences a null during object construction, which the runtime
reports as a crash before any UI exists. Pure-function unit tests never construct this object through the
same path, so the defect reaches release with a green suite.

## Exceptions and boundaries

This is an ordering hazard, not a lifecycle bug: restructuring declarations and adding the suspension point
removes the crash, but the durable fix is to stop doing work in the constructor at all. A background
dispatcher or a lifecycle-scoped launcher does not have the inlining problem. If a state holder must
self-start, keep exactly one init block in the codebase and treat it as a review hotspot.

## Example

A release sets a new first line in the init block - a synchronous snapshot load - and every cold start
crashes on open. The previous first line was a suspending stream read, which had made the ordering hazard
invisible for many versions. The fix declares the backing state above the init block and keeps the
suspension point first; a comment at the class top records the constraint.
