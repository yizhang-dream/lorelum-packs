---
{"id":"toolchain-deploy-ops.implementation.cross-wsl-via-unc-and-literal-paths","title":"Cross the Windows-WSL Boundary via UNC and Literal Paths","stage":"implementation","tech_stack":["windows","toolchain"],"applies_when":"an agent on a Windows host must read or modify files whose authoritative copy lives inside a WSL distribution, or must keep a Windows-side copy in sync with that original","severity":"warn"}
---

## When to apply

Apply when a project lives inside a WSL distribution while the agent host runs on Windows: file reads
and edits, build-output sync, or any command that must execute with the Linux toolchain and filesystem.

## Guidance

Use two channels side by side. For file content, address the Linux tree through the distribution's UNC
mount (`\\wsl.localhost\<distro>\...`, or the legacy `\\wsl$\<distro>\...`) with the editor's
read/write tools; no shell is involved, so paths and encoding stay clean. For commands, invoke the
distribution's shell with a single-quoted command string and let the command itself do the work.

Keep non-ASCII paths as literal text in those commands. A path assigned to a shell variable can arrive
mangled across the boundary, so write the path inline or pass it as a quoted argument rather than
through a variable - quoting and expansion rules differ on the two sides. When the Linux side is the
source of truth and a Windows copy exists only for convenience, treat the copy as a build artifact:
regenerate it with the project's build command and verify it, never edit it in place.

## Why

The boundary stacks three escaping and encoding systems - the Windows API, the Linux filesystem, and
the shell that bridges them - and each has its own failure mode. The UNC channel bypasses the shell
entirely, which is why it is the reliable path for content, while garbled and mangled paths come from
the shell layer. Editing the copy makes the two sides diverge invisibly: the next build silently
reverts the edit, and any comparison between the copies stops meaning anything.

## Exceptions and boundaries

The UNC mount is for interactive, low-volume access; heavy or parallel IO belongs on the native side of
the boundary. Prefer the documented localhost mount over ad-hoc share paths, whose semantics differ. If
the project can be cloned and built on the Windows side, do that instead of crossing the boundary on
every step - fewer boundary crossings means fewer translation layers to debug.

## Example

A small web project lived in the home directory of a WSL distribution while the agent ran on Windows.
Reads and edits went through the UNC path, commands went through a single invocation of the
distribution's shell, and a non-ASCII directory name worked only when written literally - the same path
stored in a shell variable arrived garbled. A Windows-side build copy was treated as generated output:
it was refreshed by a rebuild, and the zero-diff check between the two copies remained a meaningful
acceptance signal instead of reporting hand edits.
