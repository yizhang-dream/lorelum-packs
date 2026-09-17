---
{"id":"app-delivery-ops.implementation.design-gestures-with-escape-hatches","title":"Design Gestures with Escape Hatches and Survivable Listeners","stage":"implementation","tech_stack":["web","frontend","mobile"],"applies_when":"a pointer or touch gesture is being extended with a second mode (resize, copy, select) or the element that starts the gesture can unmount or re-render mid-drag, and users report that a familiar gesture stopped responding or was swallowed by the new mode","severity":"warn"}
---

## When to apply

Apply whenever one gesture surface gains a second meaning - edge resize on a movable block, long-press on a
scrollable list, drag versus scroll - and whenever the element receiving pointer events can be removed or
replaced while the pointer is still down (column switching, list virtualization, filtering).

## Guidance

Two failure modes account for most gesture regressions:

- Do not lock the mode at pointer-down. Decide the mode dynamically from the dominant direction once the
  pointer has moved past a small threshold, and keep a cross-axis escape hatch: if the accumulated movement
  along the other axis exceeds the threshold, switch modes mid-gesture while freezing the axis the new mode
  must not alter, so a resize-turned-move does not drift in time.
- Do not keep listeners on an element that can unmount. Route pointer events at the window level and
  dispatch by pointer identifier, and render the drag preview in a container that outlives the source
  element. Skip explicit pointer capture where the platform already captures implicitly; explicit capture
  on some engines invites spurious cancellation.

Separate gesture from scroll with an activation threshold that matches the input type: a small movement
threshold for a mouse with immediate visual feedback, a sustained long-press with haptic feedback for
touch, and treat movement before the threshold as scroll intent and cancel. Mark the draggable surface as
non-scrolling, and keep scroll originating from empty space.

## Why

Hit regions are chosen by position, but user intent is revealed by motion. A mode fixed at pointer-down
commits before any evidence exists, so an edge grab followed by a horizontal move - a natural pattern -
feeds a handler that only reads the vertical axis and the gesture appears dead. Likewise, a listener bound
to the source element dies the moment the framework unmounts it during a rerender, which silently ends the
gesture even though the finger is still down.

## Exceptions and boundaries

Native platform scroll and drag conventions win over custom schemes on mobile; do not invent a gesture that
fights the OS. Provide a non-gesture path (explicit handles, buttons, keyboard) for every capability so the
escape hatch is not the only route. Threshold tuning is device- and input-specific: validate on the real
input surfaces the app supports before freezing constants.

## Example

Users report that horizontal dragging stopped working after edge-resize shipped. The resize hot zone locked
the mode at pointer-down, and edge grabs are the natural habit, so horizontal movement was consumed by a
vertical-only handler. The fix adds a cross-axis escape threshold that switches back to move and pins the
start time. In a later variant the same gesture died when the source block unmounted as the drag crossed a
column boundary, which moving the listeners to the window and rendering the preview in a separate layer
resolved.
