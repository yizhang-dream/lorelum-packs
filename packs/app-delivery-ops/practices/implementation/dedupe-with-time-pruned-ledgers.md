---
{"id":"app-delivery-ops.implementation.dedupe-with-time-pruned-ledgers","title":"Deduplicate Triggered Work with Time-Pruned Ledgers","stage":"implementation","tech_stack":["mobile","web"],"applies_when":"a scheduled or retried side effect such as a reminder, notification, or catch-up job can be triggered again by restarts, reconnects, or repeated checks, and duplicate execution would be user-visible","severity":"warn"}
---

## When to apply

Apply when implementing or reviewing anything that fires on a schedule or on retry: local
notifications, reminder engines, boot-time rescheduling, queue flushes, or catch-up after the app
was closed. Also apply when a "sent" boolean already exists and the same item can fire more than
once.

## Guidance

Do not model delivery as a boolean flag. Use an event-level idempotency key built from the stable
identity of the entity plus the exact scheduled moment it represents, and store it with the time it
fired. Every path that can fire the side effect - the first attempt, retry after failure, app
refocus, reload, and boot - must consult and update the same ledger.

Prune the ledger by time, not by count: keep only entries newer than a window comfortably larger
than the longest catch-up or grace period, so local storage does not grow forever while late
deliveries still deduplicate. Write the key before or atomically with the side effect so a crash
between the two cannot double-send. Use the same key model on every platform and on the server
side if the server also sends.

## Why

A boolean cannot distinguish "this occurrence was delivered" from "the next occurrence of the same
item is still pending", so the flag either suppresses future legitimate deliveries or re-sends past
ones. Catch-up logic deliberately re-evaluates missed triggers, which is exactly when duplicates
appear, so idempotency belongs in the state model rather than in the caller's care. Unbounded local
ledgers eventually break storage budgets in the field.

## Exceptions and boundaries

A ledger pruned too aggressively re-sends old occurrences; choose the window from the product's
actual catch-up promise, and document what happens for occurrences older than it (normally:
discard). If delivery must be deduplicated across devices rather than per device, the ledger has to
live server-side or in synced state - decide that explicitly instead of assuming.

## Example

An app's reminder engine stores keys shaped task-id plus trigger timestamp together with a fired-at
time. Repeated polling, screen focus, and reload all check the same ledger before delivering, so
several check rounds produce one notification. Failed deliveries retry within a thirty-minute grace
window, missed triggers older than the window are dropped by rule, and entries older than seven
days are pruned, keeping the store small.
