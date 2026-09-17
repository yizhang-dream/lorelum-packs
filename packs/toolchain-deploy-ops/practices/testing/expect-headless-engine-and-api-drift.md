---
{"id":"toolchain-deploy-ops.testing.expect-headless-engine-and-api-drift","title":"Expect Headless Engine and API Drift in Automation","stage":"testing","tech_stack":["toolchain","windows"],"applies_when":"writing or reviewing automated tests that drive a game or simulation engine from the command line: headless smoke suites, screenshot capture steps, or assertions over the engine's console output","severity":"warn"}
---

## When to apply

Apply when scripting an engine binary for CI-style checks - smoke suites, capture or screenshot steps,
property and exit-code assertions. Also apply when upgrading the engine version, where several of these
traps re-fire at once.

## Guidance

Separate the run modes. A headless run is right for logic smoke tests, but a capture or screenshot step
can block waiting for a frame that an offscreen window never delivers with the headless flag set; run
capture without that flag, and keep the smoke command and the capture command distinct.

Do not assume an engine API keeps its names or its meaning across versions. Discover the real members at
runtime (query the object's property list) before writing to one, because a wrong name for a property
that once existed fails silently as a no-op with no error - the symptom is "my parameter has no effect"
and it is misread as a product bug.

Assert on values, not on their string form: a float formatted to text does not equal the integer-looking
string a test may expect. Measure the engine's own exit code by redirecting its output to a file and
reading the status separately, because a pipeline reports the status of its last command, so piping the
engine into a text filter measures the filter. Keep smoke runs away from development state: a smoke run
can write into the application's dev store, so reset or isolate that store, or assert only on read-only
paths.

## Why

These failures share one shape: the command completes, the output looks plausible, and the result is
wrong for a reason the output never states. The piped exit code and the string comparison produce false
green; the headless capture flag and the renamed property produce a stall or a silent no-op. All are
cheap to prevent at authoring time and expensive to diagnose later.

## Exceptions and boundaries

Querying the property list is a fallback, not an excuse to skip release notes - a major version
migration deserves a real change list. Value assertions still need an epsilon for floats. When the
engine offers a structured test mode, prefer it over scraping human-readable stdout, which is a moving
target.

## Example

A smoke suite reported an all-PASS result with exit code zero, but the status had been measured from
the text filter in the pipeline, not from the engine; rerunning with the output redirected to a file
showed the engine's real status. In the same suite a numeric assertion built with string contains
failed because the engine printed a decimal form where the test expected an integer-looking one.
Separately, a screenshot step hung until the headless flag was removed, and a newly added camera
parameter did nothing because that property had been renamed in the engine version in use.
