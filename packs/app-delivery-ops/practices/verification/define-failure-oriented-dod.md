---
{"id":"app-delivery-ops.verification.define-failure-oriented-dod","title":"Define a Failure-Oriented Definition of Done","stage":"verification","tech_stack":["web","mobile"],"applies_when":"closing a reliability or data-integrity change, where passing tests for the normal path would not prove that failures are handled safely","severity":"warn"}
---

## When to apply

Apply when a sprint or fix is meant to protect user data (sync, storage, permissions,
concurrency) and its definition of done is about to be written as "code finished and tests green".
Also apply when reviewing another agent's findings before acting on them.

## Guidance

Write the definition of done as failure scenarios, not as a list of changed files. The standard is:
prove the normal path works and actively inject the failures the system claims to survive, then
show it does not silently corrupt user state. Concretely:

- Concurrency: fire many parallel writes at the same resource and assert zero lost updates.
- Isolation: assert that one account cannot read, restore, or delete another account's data.
- Interruption: kill the process mid-edit and assert the state is still recoverable.
- Recovery: deliberately pollute data, detect it, stop the spread, restore from backup, verify.
- Time: inject a fake clock or call the engine directly instead of waiting for real time to pass.

Run these as scripted scenarios with a pass count, and record the counts in the release note. When
a reviewer reports a finding, verify each assertion against the code before fixing it - classify it
correctly (for example, cross-user reads are disclosure, not tampering) so the fix has the right
scope, and re-run the gate afterward.

## Why

Green tests written alongside the code exercise the paths the author had in mind; races, partial
writes, and permission escapes live in the paths nobody exercised. Failure injection converts
"should be safe" into demonstrated behavior, and checking reviewer claims against source prevents
fixing the wrong problem at the wrong priority.

## Exceptions and boundaries

Not every change deserves fault injection: copy changes, layout fixes, and pure functions are
adequately covered by ordinary tests. Fake-clock injection must go through the real engine entry
point, otherwise the test proves only the fake. A gate is only meaningful if it runs before release
and its failure blocks the release.

## Example

An app's reliability gate lists: one hundred concurrent writes to the same user bucket with zero
loss; last-write-wins convergence for the same record; cross-account backup listings returning
nothing; a pollute-detect-restore drill passing; and reminder idempotency across restart and missed
triggers. The reminder scenarios are driven by direct engine calls with an injected clock rather
than waiting thirty minutes, so the whole gate runs in seconds.
