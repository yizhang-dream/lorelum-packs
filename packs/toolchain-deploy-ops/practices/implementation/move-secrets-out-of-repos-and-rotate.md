---
{"id":"toolchain-deploy-ops.implementation.move-secrets-out-of-repos-and-rotate","title":"Move Credentials Out of the Repo and Rotate Exposed Ones","stage":"implementation","tech_stack":["toolchain","deployment"],"applies_when":"a credential, API key, token, or password is found embedded in a tracked script, config file, or note, or a repository is about to be published while suspect strings remain in its history","severity":"warn"}
---

## When to apply

Apply during repository renovation, before publishing or pushing a repository to a new remote, and
whenever a scan finds a literal credential in tracked content. The same procedure applies to
credentials that were already removed from the working tree but still exist in commit history.

## Guidance

Handle each credential by where it is consumed, and give each consumer one indirection:

- A shell or automation script reads the value from an environment variable whose name is documented
  in the repo (example file or README), never from a literal.
- Application code reads it from a local, ignored configuration file (dotenv-style), with a
  checked-in example that contains placeholder names only.
- Ad-hoc notes and scratch files move to a user-scoped secrets directory outside the repository;
  the repo may keep a pointer to that location but not the value.

Add or extend ignore rules for the local secret files in the same change, and confirm the ignore
rule actually matches the path used. Then verify the indirection: grep for the literal across
tracked files and across history, and run the consumer once from a clean shell to prove it still
resolves the value.

Rotation is mandatory for anything that ever entered a repository history, independently of any
history rewrite. Treat history rewriting as an optional extra, never as a substitute.

## Why

A committed value is exposed to every clone, fork, cache, and remote copy, and deletion from the
current tree does not remove those copies. Environment variables and untracked files can be rotated
and scoped per machine; hardcoded literals cannot. Relocation prevents recurrence for future
commits, while rotation is the only step that invalidates the exposure that has already happened.

## Exceptions and boundaries

An unpushed repository can still require rotation: the value may exist in local clones, backups, or
shared archives, and the same credential may be reused elsewhere. When in doubt, rotate, because
relocation alone does not invalidate any copy. Do not fix the leak by moving values into files
that are themselves tracked, such as committing the local dotenv file. CI and deploy pipelines may
inject values through the platform secret store instead of a local file. Do not rewrite history that
others may have based work on without coordinating first.

## Example

A renovation finds one API key hardcoded in a shell watch script, a benchmark script, and a
plaintext note. The fix routes the three consumers to an environment variable, a local ignored
dotenv file, and a user-level secrets directory respectively, and the keys are rotated because all
three had been committed. A follow-up grep over history confirms no copy remains in tracked
content, and a clean-shell run of the watch script proves the variable path works.
