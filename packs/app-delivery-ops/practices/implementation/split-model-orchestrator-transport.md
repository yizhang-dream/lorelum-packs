---
{"id":"app-delivery-ops.implementation.split-model-orchestrator-transport","title":"Split Model, Orchestrator, and Transport Layers","stage":"implementation","tech_stack":["web","agent-ops"],"applies_when":"the same application logic must serve several surfaces (web UI, CLI, or an agent-facing protocol server) and the implementations risk drifting apart","severity":"warn"}
---

## When to apply

Apply when one product exposes its behavior through more than one interface - for example a web app
plus a CLI plus an agent-facing protocol server - and any surface could reimplement validation,
persistence, or request handling. Also apply before adding a second surface to an existing app.

## Guidance

Cut the code along three seams and keep dependencies one-directional:

- model: pure functions for validation, normalization, and decision logic, with no I/O and no
  framework imports;
- orchestrator: exactly one module that sequences operations, applies guards, persists results, and
  resolves conflicts - the only place allowed to write state;
- transport: thin adapters (HTTP, streaming, CLI commands, protocol servers) that parse input, call
  the orchestrator, and format output.

Because the model layer is pure, it can be tested once and compared across surfaces: write a parity
suite that runs the same vectors through the web path and the CLI path and asserts identical results.
Mock the backend at the transport boundary in protocol tests so the orchestrator is exercised as a
whole without a live service.

## Why

Drift between surfaces stays invisible until a user notices the CLI accepts input the UI rejects.
Shared pure logic plus a single orchestrator removes the second copy of the rules, so a behavior fix
lands everywhere at once. One-directional dependencies also keep the model importable from every
surface without import cycles.

## Exceptions and boundaries

Very thin surfaces may skip a separate transport file when the adapter is a few lines inside the
command handler. Do not push presentation concerns (formatting, locale) down into the orchestrator to
avoid duplication - that only relocates the drift. If a surface genuinely needs different semantics,
make the difference explicit in the model layer instead of forking the orchestrator.

## Example

A scheduling app ships a web UI, a CLI, and a protocol server over one backend. The model layer
(shared vectors compared against the web implementation), the single orchestrator (guards,
persistence, dedupe, last-write-wins), and the transport (fetch plus streaming) live in separate
modules, and protocol tests spawn the server against a mock backend. When the ESM package must import
a CommonJS dependency, wrap it with the standard `createRequire` bridge in the transport layer rather
than duplicating its logic.
