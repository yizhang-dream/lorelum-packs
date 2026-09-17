---
{"id":"toolchain-deploy-ops.implementation.prewarm-and-async-heavy-ingest","title":"Enqueue Heavy Ingests Asynchronously and Prewarm Their Caches","stage":"implementation","tech_stack":["deployment","toolchain"],"applies_when":"a synchronous upload or ingest endpoint is asked to process an artifact whose parsing takes minutes to hours, or a client has already timed out on such an upload","severity":"warn"}
---

## When to apply

Apply when an ingest endpoint performs expensive derivation - OCR, layout analysis, media
transcoding, embedding - inside the request cycle, and the input can be large enough to exceed a
client or proxy timeout. Also apply when a batch of such artifacts must be ingested unattended.

## Guidance

Never make an interactive request wait for a derivation that takes minutes. Split it into two moves:

1. Make the expensive path asynchronous. Accept the upload, return immediately with an accepted
   status and a job identifier, and let a background worker parse it. Serialize the worker with one
   global ingest lock so the queue cannot multiply the load, and expose a status endpoint that the
   client polls for stage and completion.
2. Prewarm what cannot be async. When an artifact must go through a synchronous call, derive the
   expensive intermediate offline first and store it in a content-addressed cache, then upload. The
   interactive call becomes a cache hit measured in seconds, and the cache persists, so later
   re-uploads and re-runs of the same bytes are free.
3. Treat a client timeout as unknown, not failed. Before retrying or repairing, list what the
   service already has and check the job status; the server often completed after the client gave
   up, and a blind retry duplicates work or entries.

## Why

An HTTP request is a poor scheduler: the timeout budget is set by clients and proxies, not by the
work, so any derivation longer than that budget fails intermittently and at random. The expensive
work is usually content-derived and therefore cacheable: a whole-book OCR run costs hours once and
is free on every later ingest of the same file, whereas redoing it inside a request costs the same
and still risks the timeout. A single-worker queue keeps the cost bounded on one machine, where
parallel heavy parses contend for the same accelerator, disk, and memory.

## Exceptions and boundaries

Small artifacts do not need the async path - keep a synchronous route for documents that parse in
seconds so callers who want an immediate answer still get one. An async endpoint is only safe if
there is a status channel and a bounded queue; otherwise it just moves the overload out of sight.
Prewarming costs hours of accelerator time, so schedule it in a window that does not compete with
other heavy jobs (see the serialization practice), and remember the result is keyed by content: a
re-encoded or resaved file is a different key and misses the cache.

## Example

Uploading a several-hundred-page scanned book to a synchronous ingest endpoint repeatedly timed out.
Prewarming the OCR into the content-addressed parse cache first, then uploading, completed in
seconds. For batches, the same endpoint was called with the async flag: it returned an accepted
status and a job id, a background worker parsed under one lock, and progress was polled instead of
being inferred from a client-side timeout.
