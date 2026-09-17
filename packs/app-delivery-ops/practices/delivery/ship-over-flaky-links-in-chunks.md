---
{"id":"app-delivery-ops.delivery.ship-over-flaky-links-in-chunks","title":"Ship Artifacts Over Flaky Links in Chunks","stage":"delivery","tech_stack":["web","agent-ops"],"applies_when":"release artifacts or other multi-megabyte files must be transferred to a remote host over an unreliable network link, and single long-lived transfer pipes die midway or leave the server half-updated","severity":"warn"}
---

## When to apply

Apply when pushing builds to a server over an intermittent VPN, throttled tunnel, or mobile link,
where a streaming transfer can drop after minutes of apparent progress. Also apply whenever deploy
verification relies on file size or on the client's exit code.

## Guidance

Split the artifact into fixed-size chunks (a few hundred KB each is a good starting point) and send
each over its own short-lived connection. After each chunk, stat the destination and confirm its
size, so a dropped chunk is detected immediately and only that chunk is retried; a long single
stream gives no such checkpoint. After merging the chunks on the server, verify a whole-file
cryptographic hash and compare it against the local value.

Never verify a new build by byte count: unrelated builds can share the same size, so size is not
evidence of freshness or integrity - only the hash is. After the transfer, explicitly unpack the
archive server-side and run marker checks, including negative markers asserting that files deleted
in the release are actually gone (archive sync does not remove them).

Decouple long server-side work from the connection: start it detached with output to a log file,
then poll the log for completion markers, so a dropped client connection does not kill a running
deploy. Clean up stale client processes between attempts. When an attempt reports failure, probe
the log before retrying - the payload may already have been delivered and executed, and blind
retries cause duplicate deploys.

## Why

Retry plumbing for a failed stream is where most deploy time is lost: a long pipe that dies midway
discards all progress, and a hung client process can be unkillable by ordinary timeouts. Chunking
converts one fragile transfer into many small verifiable ones with a bounded blast radius, and hash
verification is the only check that actually proves the delivered artifact is the new one.

## Exceptions and boundaries

On a reliable, fast link, chunking adds overhead and a plain copy is fine; keep it for the paths
where transfers have historically failed. Chunking is not integrity: the final hash check is still
required. Per-chunk size checks catch truncation but not corruption, and they cannot detect that
the server-side merge ran on a stale chunk set - sequence the merge to require all chunks present.

## Example

A ten-megabyte build transfer that repeatedly died at random points was replaced by nine chunk
uploads over independent short connections, each size-checked on arrival, followed by a server-side
merge and a whole-file hash comparison. The transfer completes in tens of seconds and passes on the
first attempt, while release verification compares hashes because two consecutive builds happened
to have byte-identical sizes. A later attempt whose client reported failure was found, by probing
the server log, to have already deployed.
