---
{"id":"windows-agent-ops.review.independent-review-gate","title":"Gate Subagent File Changes Through an Independent Read-Only Reviewer","stage":"review","tech_stack":["agent-ops"],"applies_when":"a subagent has modified files and the result is about to be reported to the user, merged, or used as input to the next step","severity":"warn"}
---

## When to apply

Apply whenever a dispatched subagent changed files. Also apply before accepting a subagent claim of
completion that will be trusted downstream, and before merging or publishing any agent-authored
change.

## Guidance

Before answering the user, dispatch a separate reviewer subagent that is strictly read-only. Give it
three inputs: the original dispatch instruction, the executing agent report, and the list of changed
files. Read-only matters: a reviewer that can edit may silently fix or hide the very problem it is
supposed to report.

Review in this order:

1. Acceptance criteria from the dispatch instruction - check each one and demand evidence (command
   output, file and line). If the dispatch had no checkable criteria, write them before reviewing; do
   not review against impressions.
2. Diff scope - confirm there are no out-of-scope edits and no caller left broken by the change.
3. Regression and verification - re-run the stated verification commands and check that the claims
   match reality.

Verdicts: pass or pass-with-notes releases the change; fail or any blocker returns the issue list to
the executing agent for rework, followed by re-review. Cap a task at two rework rounds; after that,
escalate to the user with the open issue list instead of looping.

## Why

Authors are blind to their own assumptions; the agent that produced a change is the worst judge of
whether it met the criteria. An independent reader starting from the acceptance criteria catches scope
creep and unverified claims cheaply, before the user sees them. The gate costs one review pass and
prevents shipping a wrong change.

## Exceptions and boundaries

One-line edits, pure formatting changes, and research or monitoring tasks with no file changes are
exempt; a quick skim by the dispatcher is enough. The reviewer must not expand the task or refactor
while reviewing, and disagreements that survive two rounds are a decision for the user, not a reason
for more loops.

## Example

A coding subagent reports "done, tests pass" after changing three files. The reviewer first re-runs
the acceptance command from the dispatch, then reads the diff and finds an unrelated helper rewritten
along the way and one caller not updated. Verdict fail; the issue list goes back to the executing
agent; the second round passes and the result is reported to the user.
