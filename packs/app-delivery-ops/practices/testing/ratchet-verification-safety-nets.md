---
{"id":"app-delivery-ops.testing.ratchet-verification-safety-nets","title":"Ratchet Layered Verification Safety Nets","stage":"testing","tech_stack":["web","frontend"],"applies_when":"a codebase has outgrown review alone as regression protection, but existing violations make a strict full-repo gate impossible to enable today","severity":"warn"}
---

## When to apply

Apply when adding automated gates to a project that already contains violations (oversized files,
loose contracts, unverified rendering), so a binary pass/fail gate would fail on day one and be
disabled. Also apply when the existing test suite passes yet regressions still reach releases.

## Guidance

Layer the nets so each catches a different class, and gate on exit codes rather than on parsing
output:

- a functional suite that exits nonzero on any failure - the base net;
- ratchet checks (file size, module contracts) that compare against a committed baseline and fail
  only on new violations, letting the baseline move only in the safe direction;
- golden artifacts for output that is hard to assert directly (rendered pages, generated documents),
  captured from real output and compared on later runs.

Commit every baseline to the repo so accepted debt is explicit and reviewable in diffs. When a change
intentionally alters gated behavior, re-capture the baseline in the same commit; a baseline update
with no corresponding behavior change is a warning sign in review.

## Why

Strict gates on a dirty codebase get switched off by whoever is blocked by them. A ratchet stops new
debt immediately, keeps every existing exception visible as data, and allows the backlog to shrink on
a deliberate schedule. Exit-code gating matters because a suite that prints failures but returns zero
is not a gate at all.

## Exceptions and boundaries

A ratchet only prevents new violations; it does not by itself reduce the backlog, so pair it with a
plan for the remaining over-limit items. Golden artifacts must be regenerated deliberately and
reviewed - auto-accepting changes on every run turns the net into a rubber stamp. Keep the commit-path
gates fast; heavy suites belong in CI.

## Example

A web app adds three nets: a functional test run judged by exit code, size and component-contract
checks whose baselines live in committed JSON files, and page snapshots under test fixtures. The
first run flags only new violations, the team fixes those in the same commit, and later rendering
changes routinely re-capture snapshots as part of the change. Regressions that review missed are
caught by one of the three nets before release.
