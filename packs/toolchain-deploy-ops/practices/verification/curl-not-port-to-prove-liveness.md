---
{"id":"toolchain-deploy-ops.verification.curl-not-port-to-prove-liveness","title":"Prove Liveness with a Request, Not a Listening Port","stage":"verification","tech_stack":["deployment","toolchain"],"applies_when":"a local service is reported up or down during deploy or restart verification, or a port scan shows the expected port listening and the agent is about to conclude the service is healthy","severity":"warn"}
---

## When to apply

Apply to every "is it up?" decision for a multi-process service stack: after a restart, after the
host was near memory or commit exhaustion, and whenever a port check is the evidence being used to
report health.

## Guidance

Treat a listening socket as necessary but never sufficient. Liveness is a request that completes
inside a bounded timeout and returns the expected status - a health endpoint, an entry page, or a
version marker:

    curl -sS -m 5 -o /dev/null -w '%{http_code}\n' http://127.0.0.1:<port>/health

Check every port in the stack separately (front end and back end) and require a concrete success
value: HTTP 200, or an expected body. Do not accept "connection established" or a TCP-level probe as
proof. When a socket is listening but the request times out, the process behind it is a zombie that
still holds the port: identify the PID owning the listening socket, kill it, and restart the whole
service group through its normal launcher rather than starting only the missing piece. After the
restart, re-run the same request check on every port before reporting success.

## Why

A process can die in a half-closed state where the port stays bound and accepts connections but
nothing serves them, so the stack looks alive to every cheap check and fails only for a real caller.
That state follows abrupt terminations - memory exhaustion, forced kills, a launcher killed while
its children keep running - and it also leaves orphan children that survive a partial restart.
Requests verify the whole path (listener, worker, dependencies) in one step; port checks verify only
the kernel socket table. The asymmetry decides it: a curl costs a second, while reporting a dead
service as healthy sends the next person hunting in the wrong place.

## Exceptions and boundaries

A health endpoint that returns a static string proves the serving process is responsive, not that
its dependencies - database, model server, embedding service - are usable; when those matter,
exercise an endpoint that touches them. A service that is deliberately rebuilding may be listening
and slow to answer: raise the timeout and look for a progress signal instead of declaring it dead. A
request check says nothing about correctness of the payload, only about liveness.

## Example

After the host ran out of commit charge, the back-end process was gone while the front-end process
still held its port in LISTENING state - and a request to it timed out. The correct reading was a
zombie half-dead state, not a healthy service. Killing the stale process and restarting through the
launcher brought both ports to a 200 response within seconds; a port-only check would have reported
success the entire time.
