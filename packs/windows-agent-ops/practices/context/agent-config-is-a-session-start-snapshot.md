---
{"id":"windows-agent-ops.context.agent-config-is-a-session-start-snapshot","title":"Treat Agent Configuration as a Session-Start Snapshot","stage":"context","tech_stack":["agent-ops"],"applies_when":"an agent definition, model routing file, or prompt template is edited mid-session and the running session is expected to run on the new value","severity":"warn"}
---

## When to apply

Apply before concluding that a model-routing or prompt change "did not work", and before trusting a
configuration file as a description of what is actually running. Also apply when adding a new agent
type and expecting the current session to see it.

## Guidance

Model selection and system prompt are captured when a session starts. Editing the definition file
after that changes nothing for the running session or for the subagents it dispatches: each dispatch
record carries the snapshot taken at session start. A newly added agent type is likewise invisible to
the current session, because the type list is read once at startup.

The verification question is therefore not "did the file change" but "what did the session actually
run":

- Re-read the dispatch records from the session log and check the model field of each dispatch,
  instead of trusting what the definition file now says.
- Treat silent fallback as the default failure mode. A model reference that fails to resolve falls
  back to the parent session's model without an error, so a mistyped or unquoted value produces a
  plausible-looking, wrong result that only log inspection catches.
- Audit in bulk when routing matters: scan the whole session history and count which model each
  dispatch actually ran on, rather than spot-checking one.

Sequence the work accordingly: make the edit, start a new session, then verify from that session's own
records.

## Why

Definition files describe intent; the session snapshot is what runs. Because the fallback is silent
and the fallback target is usually a valid model, a broken reference produces no error signal at all -
the only reliable evidence lives in per-dispatch records. Assuming "edit means effective" turns a
configuration bug into an unexplained behavioral one.

## Exceptions and boundaries

Some hosts do read parts of the configuration per dispatch, so the snapshot rule applies to the fields
documented as session-scoped - check logs before generalizing to every setting. Restarting is cheap
for a session that has not accumulated context and expensive for a long one, so make definition edits
at a task boundary when possible. Do not report a routing change as verified from the file alone; a
green reading of the config is not evidence about a running session.

## Example

A routing file is corrected mid-session after a model reference was found to be malformed. The running
session keeps dispatching on the old value, and its subagents inherit the parent model. Starting a new
session and auditing its dispatch records shows the corrected value on every dispatch, which is the
first point at which the fix can be called verified.
