---
{"id":"research-ops.context.triage-cross-session-content-mismatch","title":"Triage Cross-Session Content Mismatch Before Acting","stage":"context","tech_stack":["agent-ops"],"applies_when":"an instruction or review comment arrives in a session while its paths, experiment identifiers, or terminology clearly belong to a different project than the one this session is tracking","severity":"warn"}
---

## When to apply

Apply when the content of a message does not cohere with the session's project: repository paths that
do not exist here, an experiment-numbering scheme from another codebase, or domain terms that belong
to a different effort. The signal is internal inconsistency, not disagreement about a decision - a
mismatch of world model, not a difference of opinion.

## Guidance

Detect, state, and do not execute. Name the specific mismatching evidence (path, identifier scheme,
terminology) so the sender can verify it in one read, say explicitly that you are not acting on the
instruction, and ask the intent as a single question with the plausible options: mispost, deliberate
methodology borrowing, or something else. Do not silently comply, and do not silently ignore.

If no clarification arrives, keep the non-executing state and continue the session's own work; a
repeated rerun of the same question is noise. If the content is a transferable methodology rather than
a misplaced instruction, the useful response is to map it onto this project's equivalent rules and
state the provenance of each mapping - that gives value without acting on the wrong target.

When this class of confusion is settled and work resumes, keep the batch discipline the borrowed
methodology itself recommends: one batch of work retires one important uncertainty. A session that
adopts a foreign rule set should adopt that rule too, not only the parts that justify more runs.

## Why

Acting on misplaced context produces changes in the wrong repository with the wrong identifiers, and
the damage is asymmetric: the work looks plausible, so it may survive review and pollute a line's
record. Asking costs one message; unwinding a wrong commit costs a corruption investigation. Stating
the mismatch explicitly also protects the sender, who may not know that two sessions or two projects
have been confused.

## Exceptions and boundaries

Shared vocabulary can produce false positives - if the terms are standard in both projects, verify by
looking for at least two independent mismatches before raising the flag. Do not use the mismatch as a
reason to refuse work the sender clearly intended: after clarification, proceed normally and record the
provenance of any borrowed method. Do not escalate into lecturing about the mix-up, and do not ask more
than once without new evidence.

## Example

A set of verification comments referring to another project's experiment numbering and directory layout
arrives in this session. The agent quotes the mismatching identifiers, states that it will not act,
offers the mapping from those principles to this project's equivalent discipline, and asks whether it
was a mispost. The sender replies only "continue", so the agent leaves the instruction unexecuted and
resumes its own line.
