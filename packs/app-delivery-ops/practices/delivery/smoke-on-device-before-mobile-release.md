---
{"id":"app-delivery-ops.delivery.smoke-on-device-before-mobile-release","title":"Smoke on a Device Before Every Mobile Release","stage":"delivery","tech_stack":["mobile"],"applies_when":"a mobile build is about to be published - first release of a feature, a hotfix, or a routine version bump - and the verification so far is unit tests plus a successful compile","severity":"warn"}
---

## When to apply

Apply to every mobile release, including patch releases and versions that only touch backend-connected
code. Apply before uploading the artifact, and treat it as a gate: no smoke run, no release. Do not apply
it to internal debug builds that are not distributed.

## Guidance

Run a short, mechanical sequence on a device or a long-lived emulator, and keep the artifact that proves
it:

- Clear the log buffer, launch the main activity, then count fatal entries in the crash buffer and confirm
  the process is still alive.
- Capture a screenshot for human review, since rendering and layout breakage is not visible in logs.
- Keep a dedicated booted emulator for this; probe attached devices first, because starting a second
  instance on an already-running emulator fails.
- From POSIX shells, disable MSYS-style path conversion before invoking device tooling, or argument paths
  get rewritten and commands silently target the wrong location.
- Uninstall a previously installed build when signatures differ: installing a debug build over a release
  build fails silently, leaving the old app in place and "verifying" the wrong binary.
- Verify the shipped artifact by hash, comparing the local build to the distributed copy; byte size is not
  an identifier and will collide.

## Why

Compile and unit-test green says nothing about startup: initialization order, permissions, storage, and
native dependencies only execute on a device. A crash that occurs before any test harness attaches is
invisible to the suite, and once published it reaches every user at once. The smoke run costs minutes and
covers exactly that blind spot.

## Exceptions and boundaries

An emulator smoke does not replace real-device verification for device-specific behavior such as OS
notification permissions, background execution limits, or vendor power management - schedule that pass
separately. A smoke run proves the app starts and renders; it is not a feature acceptance test. Do not skip
the smoke because the change is "small": startup regressions are the most expensive class of mobile bug.

## Example

Two consecutive versions shipped with no device run, and the second crashed immediately on open for every
user; the root cause was an initialization-order defect in a screen state holder, invisible to the green
unit suite. Adding the smoke gate - clear logs, launch, count fatal entries, screenshot - to the release
checklist catches that class of defect before upload.
