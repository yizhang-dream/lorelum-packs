---
{"id":"app-delivery-ops.correction.compare-calendar-days-by-daykey","title":"Compare Calendar Days by Day Key, Never by Millisecond Timestamps","stage":"correction","tech_stack":["web","mobile","frontend"],"applies_when":"code decides whether two calendar dates are the same day - a lookup, a highlight, a filter - and the comparison is expressed as equality of millisecond timestamps, Date objects, or raw epoch values, so one entry point reports an empty day while another shows content","severity":"warn"}
---

## When to apply

Apply whenever same-day logic is expressed as instant equality: `dateA.getTime() === dateB.getTime()`,
`Date` object equality, or an epoch number used as a map key. Also apply when a "today" lookup or row
highlight fails from one navigation path but works from another, and when dates cross a time zone boundary.

## Guidance

Normalize both sides to a canonical day key before comparing. A day key is a timezone-explicit string
(`YYYY-MM-DD` built from local components) or the platform's date-only type. Then:

- Store, index, and look up day-scoped records by day key; never key a store by raw epoch milliseconds.
- Normalize at the boundary: anything reaching "which day is this" logic is converted to a day key first,
  even when the caller obviously passes a midnight value.
- For "is this row the current one" checks, require a same-day gate *plus* the wall-clock comparison; a bare
  wall-clock test marks the same time slot on every day.
- Build keys through a single canonical helper so every writer agrees on format and zero padding.

## Why

Two instants can denote the same calendar day and still differ by milliseconds: a value captured now carries
the current clock time, while a stored value was normalized to midnight. Instant equality then reports no
match for a day that plainly exists, so the UI shows an empty day, a lookup misses, or a highlight lands
nowhere. Day-level intent needs a day-level representation; instant equality is correct only when the
question is genuinely about instants.

## Exceptions and boundaries

Ordering ("before/after") may legitimately use instants when the boundary is an instant, but day membership
still needs a day key. Do not paper over the bug by widening the comparison to a range around midnight -
that reintroduces timezone and daylight-saving edge cases. If a date-only type exists, prefer it over
hand-built strings; string keys must still be produced by the one canonical builder.

## Example

A home action navigates back to today and passes a freshly constructed timestamp carrying the clock time.
The day lookup compares it by millisecond equality against records normalized to midnight, so today is
reported empty despite having entries, while picking the same day from the calendar works. Normalizing at the
write path and comparing day keys at read time makes both entry points agree and fixes the identical latent
bug in the calendar page.
