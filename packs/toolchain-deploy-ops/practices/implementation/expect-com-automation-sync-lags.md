---
{"id":"toolchain-deploy-ops.implementation.expect-com-automation-sync-lags","title":"Expect Background COM Automation to Read Stale Content","stage":"implementation","tech_stack":["windows","toolchain"],"applies_when":"a desktop application is driven through COM automation to export or read a document, the call returns success, and the exported artifact is empty or older than what the UI shows","severity":"warn"}
---

## When to apply

Apply when a script exports documents or reads their content out of a desktop application through
COM, especially when the script also performs side effects such as uploading or submitting the
result. Also apply before trusting a sync or refresh call's return value as evidence that content is
current.

## Guidance

Treat a successful sync call as a request, not as a completion. Background automation commonly
returns success while the local cache still holds the previous revision, which surfaces as an
"empty" document: the export succeeds, the page exists, and the content is a fraction of its real
size. Force the refresh through the interactive path - start the desktop UI, navigate to the target
object, trigger the sync action there - and then verify freshness by measuring the content payload,
not by re-reading the status code.

Size is a good freshness oracle: an object whose content grows from a few kilobytes to roughly a
megabyte has genuinely been pulled, while the same order of magnitude as before means it has not.
Capture the size before and after the forced sync and assert on the change.

Before any irreversible step, verify entity identity by a stable machine-readable key. Nested
containers in productivity suites frequently expose composite identifiers whose leading segment
belongs to an inner container, so an identifier that merely looks right can address a different
object. Names collide: two similarly named notebooks, pages or course shells with near-identical
layouts are normal, and the difference lives in the numeric identifier. Resolve the identifier from
authoritative context (recent user activity, or a URL captured from the intended object) rather than
from a title match.

Handle transient COM failures by retrying the same call after a short delay instead of restarting the
pipeline, and expect direct deep links into action endpoints to be dead - drive the in-app flow
instead.

## Why

Automation surfaces report the success of the request they were given, while synchronization state
lives inside the application's own cache and sync engine. Nothing in the API contract promises that a
refresh has completed when the call returns, so a script that trusts it produces confidently wrong
artifacts - and an empty export submitted on time to the wrong destination is far worse than a loud
failure.

## Exceptions and boundaries

A document that is genuinely empty and an unsynced document look identical from the export alone, so
verify against a second signal (content size, page count, or a known string) before concluding.
Forcing UI sync requires an interactive session and is unavailable on a headless host; there, export
what the cache holds and label the artifact as possibly stale. Identity checks catch identifier
mistakes, not permission or account mistakes.

## Example

An export script publishes a notebook page to PDF and finds the page empty. The sync call returned
success; only after launching the desktop client, navigating to the page and triggering a manual sync
does the content grow from 5 KB to 1.1 MB and the export become correct. The same run confirms
identity first: two identically named course shells exist, one of them an empty duplicate, and the
submission target is chosen by the numeric identifier from browsing history rather than by title.
