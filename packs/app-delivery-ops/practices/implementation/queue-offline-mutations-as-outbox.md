---
{"id":"app-delivery-ops.implementation.queue-offline-mutations-as-outbox","title":"Queue Offline Mutations in an Outbox","stage":"implementation","tech_stack":["mobile","web"],"applies_when":"an app edits user data against a server and a failed request currently rolls the local state back to the server value, losing the user's action","severity":"warn"}
---

## When to apply

Apply to any client that performs user-initiated mutations against a remote store and handles a
failed request by refetching, with the local edit silently disappearing. Also apply when adding
offline support to a sync-based app.

## Guidance

Give the client an outbox and keep its boundary narrow. User-initiated mutations - completing an
item, skipping it, small edits - enter the queue and must succeed locally the moment the user acts:
optimistic UI, no rollback, no spinner, no error toast for being offline. Anything the user did not
directly initiate (imports, regenerations, whole-record replacement) stays out of the outbox.

Because the upload protocol is a full-document put, enqueue a full local snapshot overwrite rather
than a per-item operation queue. Flush on reconnect and on app start. When a flush fails, render
from the local snapshot; if the failed flush is followed by a refetch, the fetch can overwrite the
queued state and erase the offline work. Serialize the document write and the upload through one
mutex so a background flush cannot interleave with rapid taps. Clear the queue on logout so one
account's mutations cannot flush into another account's bucket.

Keep conflict semantics as they already are - the outbox exists to prevent loss, not to redefine
merging. Expose a minimal sync status (synced, syncing, pending) and do not surface HTTP codes,
retry counts, or queue depth.

## Why

Rolling back a failed write treats a transport failure as if the user changed their mind; the
action is gone and the user may not notice until later. Separating "do not lose mutations" from
"how two versions merge" keeps the new machinery small enough to reason about and test, and
full-document snapshot enqueue reuses the existing upload shape instead of inventing an operation
log.

## Exceptions and boundaries

Deletions can stay outside the outbox when delete plus tombstone already has correct rollback
semantics. The outbox does not guarantee that queued local edits win conflicts; the existing
last-write-wins rules still apply, and a gate should cover offline edit versus concurrent server
edit. A queue with unbounded growth, no logout clear, or no expiry is a data-leak path between
accounts - treat those as requirements, not hardening.

## Example

A mobile client previously refetched after a failed upload, so a check-in made offline vanished.
The outbox now stores the local snapshot on failure: the check-in appears immediately and the row
shows the pending state; the queue flushes when connectivity returns, and last-write-wins
convergence handles the server's concurrent changes. Killing the process before reconnect still
leaves the pending state intact, and logging out empties the queue.
