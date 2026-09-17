---
{"id":"toolchain-deploy-ops.verification.verify-live-after-deploy","title":"Verify the Live Artifact Before Retrying a Deploy","stage":"verification","tech_stack":["deployment","toolchain"],"applies_when":"a deploy command run over an interactive remote shell appears to hang or returns without output, and the agent is about to retry it or report it as failed","severity":"warn"}
---

## When to apply

Apply after any deploy whose completion is ambiguous: the remote command printed nothing, the shell
never returned a prompt, or the session was interrupted while a mutating step was in flight. Also
apply before re-running a deploy step that consumes an artifact, and before reporting to the requester
that a deploy failed.

## Guidance

Treat the live system's observable state as the source of truth, not the shell's output or exit
stream. Fetch the deployed entry document or a version marker and confirm a newly added asset resolves,
before deciding anything and before retrying.

Prefer non-interactive remote execution for deploys: no pseudo-terminal allocation, batch-mode
authentication, and remote output redirected to a file that is pulled back and read afterwards. That
separates "the command finished" from "the client stream finished" and keeps login banners out of the
parsed output. Never re-run a mutating deploy blindly - many deploy steps move or consume an uploaded
archive, so a second run fails with a missing-file error that looks like a new problem while the first
run had already succeeded. When the live check shows the old generation, the deploy genuinely failed;
read the remote log file instead of guessing from the silence.

## Why

An interactive stream is not a completion channel. Banners, login notices and shell hooks share the
same channel as the command output, and a client that stops reading can leave the local wrapper waiting
after the remote command already exited. The deploy steps themselves are usually destructive and
non-idempotent - unpack-and-replace, permission changes, deletion of the uploaded archive - so retrying
converts a successful deploy into a confusing partial state and a wrong incident report.

## Exceptions and boundaries

If the deploy tool reports a real error with a non-zero status and the live check confirms the old
generation, treat it as failed and inspect the log. A live check proves the entry point serves the
expected generation; it does not prove every asset and sub-application is correct, so check the
specific new artifact as well. Do not run a rolling or multi-stage deploy behind an ambiguous stream -
the ambiguity compounds and the check can no longer distinguish stages.

## Example

A deploy over an interactive shell appears to hang after printing a long banner. The operator assumes
failure and re-runs the pipeline, which fails with "No such file" because the first run had already
unpacked and removed the archive. Fetching the live entry page shows the new version marker and the new
asset resolves: the first deploy had completed. The reliable pattern is non-interactive execution,
remote output captured to a file, and a live fetch as the success gate.
