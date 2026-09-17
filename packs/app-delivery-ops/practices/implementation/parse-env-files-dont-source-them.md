---
{"id":"app-delivery-ops.implementation.parse-env-files-dont-source-them","title":"Parse Environment Files Instead of Sourcing Them","stage":"implementation","tech_stack":["agent-ops"],"applies_when":"scripts or test harnesses load a .env-style file whose values are edited by hand or generated from a template","severity":"warn"}
---

## When to apply

Apply whenever a shell script, test harness, or deployment step reads a .env-style file that people
edit by hand or that is generated from a template. Also apply when a previously working script starts
failing on a line someone just added to that file.

## Guidance

Treat the env file as data: open it, strip comments and blank lines, split each `KEY=VALUE` at the
first `=`, and export the variables into the process yourself. Read it with a real parser in the
language running the harness, and raise a clear error naming the offending line when a line is
malformed.

Never hand the file to the shell (`source`, `.`, or `eval`). Values with angle brackets, spaces,
`$`, backticks, or unquoted parentheses are common in config files and are not shell-safe; sourcing
turns them into syntax errors or, worse, command substitutions. When the harness is Python, parse
with a small helper and pass values to child processes through the environment mapping rather than
through an interpolated shell string.

## Why

Shell sourcing executes the file as code, so any value that happens to be valid shell syntax also
runs. A template placeholder that looks natural in a config file can abort the entire harness before
any test executes, and the error points at the shell rather than at the file that caused it. Parsing
keeps configuration in the data domain and turns malformed input into a clear, line-numbered failure.

## Exceptions and boundaries

If a file is deliberately a shell script (exports plus logic), sourcing is fine - name it as such so
no one stores bare values in it. Do not write secrets into process argv or logs while parsing; keep
them in the environment or in a file passed by path. If your files rely on full dotenv semantics
(quoting, multiline values, expansion), use a maintained parser instead of a hand-rolled one.

## Example

A local test harness sources the app's env file to log in a test user. A newly added display-name
variable whose value is wrapped in angle brackets makes bash abort with a syntax error before any
test runs. Rewriting the harness to parse the file with Python - the pattern already used elsewhere
in the project - and inject the variables via the child environment fixes it and keeps working as
more placeholder-shaped values are added.
