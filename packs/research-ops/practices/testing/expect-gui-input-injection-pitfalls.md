---
{"id":"research-ops.testing.expect-gui-input-injection-pitfalls","title":"Expect GUI Input-Injection Pitfalls","stage":"testing","tech_stack":["simulation","agent-ops"],"applies_when":"an automated agent must drive a native desktop GUI - start a run, click a control, type a value - and the injected screen input appears to have no effect","severity":"warn"}
---

## When to apply

Apply when automation injects keyboard, mouse, or scroll events into a desktop application,
especially from an agent host whose own terminal or editor raises itself between steps. Also
apply when a GUI run "did nothing" and the first suspicion is the application itself.

## Guidance

Treat focus as a contended, system-wide resource. Every command the agent runs may raise a
window, so injected events land on whatever is foreground at that instant. Run the whole
interaction from a single detached script that takes focus once, confirms the target window is
foreground, then injects - rather than interleaving injections with other tool calls.

Scale coordinates for display DPI before injecting: under a non-100% display scale the
physical pixel grid differs from the logical coordinates most APIs report, so the injection
process must be DPI-aware or every click lands at the wrong place. When the application offers
a real control for the effect you need - a speed toggle, a spinbox, a menu entry - prefer
clicking it over synthesizing keystrokes: self-drawn inputs often reject injected typing
outright, while native widgets accept it.

## Why

Injected input goes to the system-wide foreground window, not to the window the script
intends. Nothing reports an error when it lands elsewhere: the events vanish into another
process, producing a silent no-op indistinguishable from an application bug. DPI mismatch adds
a second silent failure - syntactically valid clicks at plausible-looking coordinates that
miss their target - and retries cannot fix either cause.

## Exceptions and boundaries

Headless or offscreen modes avoid focus contention entirely and are preferable whenever the
application supports them. Do not try to make injection reliable for a window that a competing
process keeps stealing focus to; restructure the run instead of adding retries or longer
sleeps.

## Example

An agent "drove" a simulation GUI while its own shell raised a terminal between steps; every
capture came back static. Moving the entire interaction into one detached, DPI-aware script
that focused the window once and clicked the built-in speed control instead of typing into a
self-drawn field produced the expected recording on the first attempt.
