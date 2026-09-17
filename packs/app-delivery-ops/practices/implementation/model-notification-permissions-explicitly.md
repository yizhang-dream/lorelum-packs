---
{"id":"app-delivery-ops.implementation.model-notification-permissions-explicitly","title":"Model Notification Permissions as an Explicit State Machine","stage":"implementation","tech_stack":["mobile"],"applies_when":"app behavior depends on an OS-level permission the user can deny or that silently blocks delivery, such as notifications or exact alarms, and asking for it casually leaves the feature failing silently","severity":"warn"}
---

## When to apply

Apply when implementing a feature that depends on an OS permission - notifications, alarms,
background execution - on a platform whose system prompt is rate-limited, can be silently
suppressed, and whose users frequently deny without understanding why they are being asked.

## Guidance

Model the permission as a state machine with persisted facts, never as a local boolean:

- Explain in-app first, and trigger the system prompt only from an explicit user gesture on a
  button such as "enable reminders". A prompt at launch has no context and is often denied or
  swallowed by the OS with no visible result.
- Persist whether the prompt has been shown, and spend the system prompt budget deliberately;
  some platforms allow only a couple of prompts per install before permanently denying.
- Read the effective state from the API that folds both the modern runtime permission and the
  older global toggle into one answer, and recompute it on resume so returning from system
  settings takes effect immediately.
- When the state is permanently denied, the only remaining path is a deep link to the system
  settings screen; show it instead of re-prompting.
- Treat silent platform-level blocks as first-class states: a notification channel disabled,
  exact alarms downgraded, battery optimization or vendor auto-start managers suppressing
  delivery. Surface guidance for these in the same flow.

Ship a self-check screen listing each gate as a row (global toggle, permission, channel, exact
alarm, battery) with a link on failing rows, so a user report of "no notifications" can be
diagnosed from a screenshot instead of guesswork.

## Why

Permission prompts are a budgeted interaction with the user and the operating system; asking at
the wrong moment burns the budget and produces a denied state that is hard to recover from. A
boolean misreads platform versions and settings toggles, and guards that silently return when
permission is missing turn the whole feature into an invisible no-op. The state machine plus a
self-check page converts an unreproducible field report into an enumerable set of causes.

## Exceptions and boundaries

Do not nag on every launch; once the state machine knows the prompt was spent, route to settings
instead. On platform versions where the permission is granted at install time, the flow still
applies for channel and battery states but no prompt should be shown. The state machine cannot
guarantee delivery against vendor background restrictions - be explicit with users about what is
confirmed versus assumed.

## Example

A reminder feature was requested only when one particular screen opened, so many users never saw
the prompt and notifications silently never arrived; a first fix that asked at launch was still
swallowed by some ROMs. The final flow shows an in-app explanation, triggers the system prompt
only from its button, records that it was asked, reads the effective toggle on resume, deep-links
to settings after permanent denial, and offers a self-check page whose rows show which gate is
blocking delivery.
