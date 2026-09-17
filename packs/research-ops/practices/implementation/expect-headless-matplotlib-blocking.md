---
{"id":"research-ops.implementation.expect-headless-matplotlib-blocking","title":"Expect Headless Matplotlib Calls to Block","stage":"implementation","tech_stack":["research","ml"],"applies_when":"an experiment or plotting script hangs with no error under a headless or batch run (Agg backend, no display), or a plotting call never returns and the process must be killed by timeout","severity":"warn"}
---

## When to apply

Apply when running plotting or visualization code written for an interactive session inside a batch
job, CI step, or any display-less environment. Apply specifically when a run prints progress and
then stalls, or when a timeout kills a process that was apparently still working.

## Guidance

Expect waiting primitives to block forever without a display. `plt.pause(0)` is the common trap: the
default event-loop implementation treats a non-positive timeout as infinite, and a non-interactive
Agg canvas has no events that would ever end the loop, so the call never returns. Patch it to a
small positive interval, or replace the interactive call with `savefig` and direct rendering - for
automated runs, select the non-interactive backend explicitly and save figures rather than relying
on `show()`.

Remove environment ambiguity before the run: execute under the project's dedicated virtual
environment with an explicit interpreter path, not the machine-default interpreter, which is often a
different distribution carrying different versions of the plotting stack. Confirm the failure
mechanism before patching - a hang can also come from network waits or data loading - and verify
completion by checking that the expected output files exist and are non-empty.

## Why

Interactive plotting assumes a GUI event loop pumping events. Headless backends implement a stub
loop, so functions that wait for user or window events either return immediately or, when the
timeout is interpreted as zero-means-infinite, spin forever. The symptom is an opaque hang: no
traceback, no partial output, and time lost to a run that looked healthy until the timeout. An
interpreter mismatch compounds it by making the bug appear or disappear with the environment chosen,
which invites misdiagnosis.

## Exceptions and boundaries

Interactive calls are fine when a human is watching a windowed session - do not strip them globally.
Prefer a targeted patch over a global pipeline timeout: a timeout also truncates legitimate long
computations and hides which call was blocking. Version differences between the project environment
and a global interpreter are a separate diagnosis; fix the interpreter choice rather than pinning
packages to match the wrong one.

## Example

A tutorial script adapted for a batch run stalls after printing its first progress line and is killed
by a timeout with no error output. The blocking call is `plt.pause(0)` under the Agg backend.
Replacing it with a small interval and re-running under the project's virtual environment completes,
and the expected figure files appear with non-zero sizes; the machine-default interpreter, by
contrast, carried different library versions and is no longer used for these runs.
