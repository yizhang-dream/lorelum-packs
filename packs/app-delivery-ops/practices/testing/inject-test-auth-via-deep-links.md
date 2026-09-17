---
{"id":"app-delivery-ops.testing.inject-test-auth-via-deep-links","title":"Inject Test Auth Through One-Time Deep-Link Codes","stage":"testing","tech_stack":["mobile","agent-ops"],"applies_when":"an end-to-end test on a device or emulator needs a logged-in session with real data, browser-driven form login is impractical or flaky, and the test must exercise the production authentication path rather than a bypass","severity":"warn"}
---

## When to apply

Apply when automating a mobile flow that starts behind authentication, when manual login on the test device
is too slow to repeat, or when a test needs the real data of a dedicated test account. Also apply when
preparing the test device image itself.

## Guidance

Mint a session the same way a real login would, but start it from the test harness:

- The server writes a single-use, short-lived code for the nominated test account into the store the auth
  service reads.
- The test opens the app's custom-scheme deep link carrying that code; the app exchanges it through the
  normal token exchange, so every downstream request uses the production path and real data.
- After exchange the app lands on its normal authenticated screen, and the rest of the end-to-end flow runs
  unmodified.

Gate the channel to debug builds and keep the code single-use and expiring; never mint codes for
production accounts. If the harness is a desktop driver instead of a device, prefer whichever entry point
the app already gate-keeps rather than adding a new bypass.

When provisioning the test device, download large SDK or system images with resumable transfers (a bounded
retry loop around a byte-range resume flag) instead of a package manager, whose big downloads die mid-flight
on unstable links and leave a corrupt image.

## Why

Driving the login UI from a test client is brittle and slow, and a stub session that skips the exchange
diverges from production exactly where bugs live - token handling and data scoping. A one-time code keeps
the test on the real path while removing the UI interaction, and the short lifetime plus debug gating keeps
the injection channel from becoming an attack surface in shipped builds.

## Exceptions and boundaries

A debug-only login screen is an acceptable alternative when the app already ships one; the point is that the
channel is unavailable in release builds. Do not reuse codes across test runs or accounts, and do not leave
test tokens in shared state. This mechanism authenticates a test harness - it is not a user-facing feature
and must not appear in release permissions or manifests.

## Example

A smoke or end-to-end run needs an authenticated session with real data. The harness asks the server for a
short-lived code, fires the app's custom-scheme link with that code, and the app performs its usual exchange,
landing on today's view with real content. The whole flow runs without touching the login form, and the
handler is compiled into debug builds only.
