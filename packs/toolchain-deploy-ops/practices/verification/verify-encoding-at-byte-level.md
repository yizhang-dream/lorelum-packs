---
{"id":"toolchain-deploy-ops.verification.verify-encoding-at-byte-level","title":"Verify Text Encoding at the Byte Level, Not in the Terminal","stage":"verification","tech_stack":["windows","toolchain"],"applies_when":"text read from a file or from command output appears as mojibake, and a conclusion is about to be drawn about whether the file content itself is corrupted","severity":"warn"}
---

## When to apply

Apply whenever non-ASCII text looks wrong on screen - terminal output, log tails, scripted replacements
- and before reporting that a file, a tool, or a pipeline has an encoding bug.

## Guidance

Treat the terminal as a lossy display layer, not as evidence. Re-read the file through a channel with a
declared encoding (the editor's file read, or an API that returns decoded text) and then check the
bytes: decode as UTF-8, assert the expected characters are present, and count code points. Run that
check inside the tool that owns the file format, for example a small Python script, rather than through
a chain of pipes.

When a batch text replacement is the suspect, inspect the mode of the replacement tool. A stream editor
run in character mode can re-encode multi-byte text that arrived from the command line as bytes, turning
correct text into double-encoded garbage. Prefer a replacement script that opens files with an explicit
UTF-8 encoding, or a pure byte-mode replacement, and re-verify the result with the same byte-level
check. Never verify with a command that truncates by bytes (`cut -c`), because it can split multi-byte
characters and manufacture mojibake of its own.

## Why

Three layers can garble text - the file, the tool that edited it, and the terminal that displays it -
and only the first is a real bug. Terminal echo depends on the console code page, and piped output
depends on the reader's locale; both can show garbage for perfectly valid UTF-8. Byte-level verification
is the only check that distinguishes "the file is wrong" from "my view of the file is wrong", and it
catches the subtler real failure: content silently double-encoded by the tool that rewrote it.

## Exceptions and boundaries

A byte-level check proves encoding, not intention - it passes on valid UTF-8 that is still the wrong
text. When the file must interop with an older consumer, the target encoding is a requirement rather
than a bug. Terminal output is fine as a first hint; it must not be the finding.

## Example

A terminal showed a batch replacement of documentation cross-references as mojibake, suggesting the
replacement had corrupted the documents. Reading the files through the editor showed the expected text.
A Python byte-level check then separated the two cases: one file was intact and merely displayed wrong,
while another was genuinely double-encoded, because the stream editor had been run in character mode
with non-ASCII arguments on the command line. The fix was a replacement script that opens files with an
explicit UTF-8 encoding, with the same byte-level check kept as the acceptance step.
