---
{"id":"app-delivery-ops.verification.capture-headless-electron-deterministically","title":"Capture Browser Renders Deterministically","stage":"verification","tech_stack":["web","frontend","agent-ops"],"applies_when":"an automated browser or Electron driver is used to capture full-page screenshots or DOM evidence of a locally served web app, and captures hang, abort, or differ between runs","severity":"warn"}
---

## When to apply

Apply when driving a real browser engine (Electron, headless Chromium, an automation host) to
capture screenshots or interactive evidence of a locally served web app for verification or visual
regression. Also apply before concluding that a page is broken because an automated capture failed.

## Guidance

Build the capture as an explicit, isolated recipe; every item below fails silently or with an
unhelpful error when omitted:

- Force the driver to ignore any system or VPN proxy for loopback addresses. A machine-wide proxy
  can black-hole localhost requests for the browser while command-line HTTP clients still work.
- Intercept and block third-party font/CDN origins and the service worker script. Foreign origins
  that are unreachable keep the load event pending; a newly registered service worker can reload
  the page mid-navigation and abort the load.
- Register exactly one request-interception listener per session; route all filters inside that one
  callback. A second listener can stall the driver's event loop, after which timers never fire.
- Use a fresh, disposable session partition instead of the real user profile, so app cookies and
  caches are neither read nor overwritten.
- Keep the capture window shown, and inject a stylesheet that disables animations and transitions;
  a hidden window pauses the compositor and produces intermediate or blank frames.
- Seed auth and demo data deterministically (mint a session cookie, write storage keys, reload)
  rather than clicking through login flows.

## Why

An automated capture is a verification instrument: if its environment is uncontrolled, a failure
says nothing about the app and a success may not reproduce. The failure modes above are silent -
hangs, blank frames, aborted loads - and a trivial HTTP probe passing is not evidence that the
browser path works, because the two do not share the same network and rendering stack.

## Exceptions and boundaries

Do not block service workers when the behavior under test is the service worker itself; use a
separate profile and assert on its lifecycle instead. Disabling animations is fine for layout and
content evidence but invalidates any check of animation timing. Captures taken this way are
evidence artifacts, not deployment inputs; keep them out of the repository and the release bundle.

## Example

A capture script launches the app in Electron with proxy bypass for loopback, blocks the external
font and service worker origins with one request listener, uses a throwaway partition, injects an
animation-killing stylesheet, mints a session cookie, and seeds demo storage before reloading once.
Full-page screenshots of every theme and viewport are then produced reliably, where the same script
without the proxy bypass and single-listener rule hung with no output.
