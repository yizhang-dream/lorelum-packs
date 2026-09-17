---
{"id":"toolchain-deploy-ops.planning.reuse-and-persist-rescue-scripts","title":"Search for Prior Tooling Before Building, and Commit Rescue Scripts","stage":"planning","tech_stack":["agent-ops","toolchain"],"applies_when":"an operational or tooling request arrives in a multi-session project and the agent is about to design or rebuild a utility that a previous session may already have produced","severity":"warn"}
---

## When to apply

Apply at the start of any operational or tooling request on a long-lived project: a service to stand
up, a data-moving script, an access or sharing mechanism, a recovery procedure. Also apply whenever a
quick script is written under time pressure to unblock the current task.

## Guidance

Search the project's own history before designing anything: the operations runbook's procedures and
trap tables, the progress or decision log, and the script and service directories in the repository. If
an equivalent exists, reuse it - read it, run it, extend it in place. If it is missing, build it and
commit it in the same effort.

Commit rescue scripts too, even one-off ones, labelled as one-off with the date and the situation that
produced them. Cross-link the artifact from the runbook or the operational memory note so the next
session finds it by search rather than by luck. A working tool must not exist only inside one chat
session.

## Why

Multi-session, multi-tool projects lose session-only artifacts by construction. When the tool is gone,
the next session either rebuilds it or offers a degraded substitute - an expiring static snapshot, a
dead link - and the requester has to explain the previous solution all over again, which is the exact
cost technical reuse was supposed to remove. Committing the tool converts one session's work into a
durable asset and makes the reuse step cheap enough that the next agent actually performs it.

## Exceptions and boundaries

Never commit credentials, tokens or machine-specific configuration; parameterize them through the
environment or a local ignored file. Throwaway probes with no reuse value may stay out of the
repository, but a script that unblocked a real task usually has reuse value, so committing is the
default. A runbook entry pointing at a script that no longer exists is worse than no entry, so update
the pointer whenever the script moves or is retired.

## Example

A real-time sharing page had been built in an earlier session but never committed. Weeks later a
re-login request could only be answered with an expiring image and a link that no longer worked, and
the requester asked why the existing tool was not reused. The response was to search the runbook,
progress log and script directories first, restore the service into the repository, and record its
entry point and one-off nature in the runbook so the next session starts from the tool instead of from
zero.
