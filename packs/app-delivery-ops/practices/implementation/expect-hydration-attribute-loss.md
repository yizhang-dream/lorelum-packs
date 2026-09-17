---
{"id":"app-delivery-ops.implementation.expect-hydration-attribute-loss","title":"Expect Hydration to Strip Attributes Written to the HTML Root","stage":"implementation","tech_stack":["web","frontend"],"applies_when":"a theme, locale, direction, or class attribute is placed on the document root before hydration (SSR markup, an inline head script, or a plugin) and deep-linked or hard-refreshed pages render with the attribute missing while the home page looks correct","severity":"warn"}
---

## When to apply

Apply when root-level presentation state (theme, color scheme, locale, text direction, a root class) is
written imperatively - by an inline script, SSR markup, or a plugin - and something looks wrong only on
direct navigation: a deep link, a bookmark, or a notification target. The signature is that the home page
looks fine and masks breakage on every other route.

## Guidance

Assume any attribute mutated outside the framework's render output is lost when hydration takes over, and
build three layers:

- Server-render a valid default on the root element so the first paint is already correct.
- Add a client component mounted at the root that re-applies the attribute in a layout effect, before
  paint, covering the hydrated tree on every route.
- Keep the pre-hydration inline script only to avoid a flash, never as the source of truth.

Verify by loading each deep route directly with a fresh profile, not by navigating from the home page -
client-side navigation keeps the already-applied attribute and hides the bug. When the framework manages
the attribute itself, feed it through framework state instead of a side channel.

## Why

Hydration reconciles the DOM against the client render. Attributes the server markup carried but the client
render does not produce can be discarded or overwritten, and nothing in the framework guarantees survival
for imperatively written root attributes. A page that repairs itself in an effect after load appears healthy
while all other pages, which lack that repair, render with an empty theme or default direction.

## Exceptions and boundaries

Attributes owned and rendered by the framework should not be double-written by a manual effect; pick one
owner. For fully client-rendered apps the failure mode differs (no hydration pass), though late attribute
writes still cause a first-paint flash. Do not fix this by disabling hydration or by suppressing the
hydration warning - the warning is describing real DOM drift.

## Example

A theme attribute is written by an inline script and by SSR on one page only. Opening a secondary route
directly leaves theme variables unset: the header gradient disappears, text renders white on white, and
call-to-action buttons look transparent, so users report "editing is broken" although saving works. The home
page's own effect had been masking the issue. The fix is an SSR default plus a root-level effect component,
with the inline script retained to prevent flashing.
