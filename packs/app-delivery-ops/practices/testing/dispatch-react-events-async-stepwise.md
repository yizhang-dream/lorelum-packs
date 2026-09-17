---
{"id":"app-delivery-ops.testing.dispatch-react-events-async-stepwise","title":"Dispatch Synthetic Events Asynchronously and Stepwise","stage":"testing","tech_stack":["frontend","web"],"applies_when":"a component behavior is tested by dispatching synthetic pointer, mouse, or touch events instead of driving a real browser, and the test passes or fails in a way that does not match observed behavior in the running app","severity":"warn"}
---

## When to apply

Apply when building or debugging an automated harness that dispatches synthetic events: drag and drop,
long press, hover, or anything whose correctness depends on rendering, unmounting, or layout happening
between the steps. Also apply when a harness reports data loss or phantom failures that the real app does
not reproduce.

## Guidance

Drive the component the way the browser would, one step at a time:

- Dispatch each event in its own task, awaiting a frame or tick between steps and reading the DOM after each
  one. React batches synchronously dispatched bursts, so a pointer-down/move/up sequence fired in one block
  produces no intermediate render and no unmount - the exact timing the test is supposed to exercise.
- Give fixtures real current timestamps. A record carrying a future update time loses last-write-wins
  against the server, whose response merge then overwrites the new write and looks like data loss.
- For elements detached from the document, synthetic events do not bubble out of their own subtree; when the
  node is disconnected, dispatch on the document body or reattach it first.
- Isolate subtests that share a per-user data bucket: parallel writes get merged back into one another, so
  scope each subtest to its own date or filter the queries to the subtest's column.

Assert on state that only the real sequencing can produce (position, count of mounted nodes, stored values)
and screenshot the result for a human check when the bug is visual.

## Why

The harness is only useful if its event timing resembles the browser's. Synchronous bursts execute inside a
single batch, so the component never renders, never unmounts the source element, and never passes through
the state the bug lives in - the test goes green while the defect remains. Fixture timestamps and shared
buckets are the other two silent corruptions: they make the harness blame the code for artifacts the harness
itself created.

## Exceptions and boundaries

Pure functions do not need stepwise dispatch; test them directly. When a real browser or device driver is
available, prefer it for gesture behavior, and keep the synthetic harness for regression coverage. Stepwise
timing makes suites slower - keep the stepwise cases focused on ordering-sensitive behavior instead of
applying the pattern everywhere.

## Example

A drag harness fired pointer events back to back and reported that the drop never completed, while manual
testing worked. The actual cause was that the source element unmounted as the drag crossed into another
column, and only real event spacing reproduced it; dispatching the sequence across separate tasks with a
frame between steps exposed it. In the same harness, fixtures carried future timestamps, so the server's
merge overwrote each new write and the run looked like the app was losing records.
