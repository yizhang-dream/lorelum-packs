---
{"id":"windows-agent-ops.planning.scheduled-task-start-boundary-is-first-fire-date","title":"Scheduled Task StartBoundary Is a First-Fire Date","stage":"planning","tech_stack":["windows","git-bash"],"applies_when":"registering or repairing a Windows scheduled task with a daily or weekly trigger and choosing a StartBoundary for automation that should already be live today","severity":"warn"}
---

## When to apply

Apply when planning recurring automation through Windows Task Scheduler - nightly jobs, periodic sync
or switch tasks - and whenever a task was registered successfully but produced no run at the expected
window.

## Guidance

`StartBoundary` is not merely a time of day: its date is the first date on which the trigger is live.
If the boundary is set to tomorrow while the task is registered today, the whole window in between is
skipped silently - the scheduler neither runs it early nor catches it up. Two habits prevent the gap:

- Set the boundary to today (or to now) when the task should be active immediately. Reserve a future
  date for automation that is deliberately staged.
- After registering, either run the task once manually or set `StartWhenAvailable` in the task XML.
  The manual run gives an immediate signal that the action and its interpreter work; the flag lets the
  scheduler run a missed window at the next opportunity instead of dropping it.

Register from PowerShell, not from Git Bash. A Git Bash invocation of `schtasks /create ...` has its
slashes rewritten into paths (`/create` becomes `C:/Program Files/Git/create`), so the switches never
reach the tool as written.

## Why

The trigger's boundary date is a lower bound on the trigger's activation. Nothing in the registration
output flags the gap, and `LastRunTime` continues to show the manual test run, so the task looks healthy
while the automation is dormant. Data written by the job simply stops advancing, and the failure is
noticed days later - often as "the source never updated" rather than "the schedule never fired".

## Exceptions and boundaries

A deliberate future boundary is correct when a rollout is staged; document it so the dormancy is not
misread as a bug. `StartWhenAvailable` compensates for missed windows caused by downtime, but it is not
an alarm: verify after the first real window by reading `LastRunTime` and `NextRunTime` from
`Get-ScheduledTaskInfo` rather than trusting the registration response.

## Example

A nightly switch job is registered at midday with both triggers set to the following day. The first
evening window fires nothing, and the task's last run time stays at the manual test from registration
time, leaving the target state unchanged overnight. Setting the boundary to the registration day fixes
the same-day window; a manual run plus `StartWhenAvailable` covers the rest.
