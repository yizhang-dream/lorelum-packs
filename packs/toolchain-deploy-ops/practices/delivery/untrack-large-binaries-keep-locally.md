---
{"id":"toolchain-deploy-ops.delivery.untrack-large-binaries-keep-locally","title":"Untrack Large Binaries but Keep Them on Disk","stage":"delivery","tech_stack":["toolchain","agent-ops"],"applies_when":"a repository tracks multi-megabyte binary assets such as media files, model weights, or generated outputs that inflate pack size and clone time, while the same machine still needs those files to keep working","severity":"warn"}
---

## When to apply

Apply when repository hygiene work finds large tracked blobs, or when clone and fetch cost has grown
out of proportion to the source code. Typical triggers: media assets, model checkpoints, packaged
build outputs, and generated archives that were committed alongside code.

## Guidance

Decide per file what the repository must provide, not what the machine happens to have:

- If the file is a local working asset that consumers fetch, download, or regenerate separately,
  remove it from tracking while keeping the working-tree copy: stage the removal with the cached-only
  form of the removal command so the file stays on disk. Add an ignore rule in the same change so it
  does not reappear as untracked noise.
- If other clones genuinely need the asset, move it to an artifact channel (release assets, object
  store) and record the fetch step in a tracked script or document; only then may history be
  rewritten or the blob deleted.
- For media that builds never read, untracking alone is the whole fix; do not delete the file.

Record the outcome in the repository: where the asset now lives, how to regenerate or download it,
and what still depends on the local copy. Measure before and after - pack size, tracked file count,
clone time - and land the untracking as one isolated change rather than mixed with feature work.

## Why

Every clone, fetch, and CI checkout pays for tracked blobs, while large binaries produce useless
diffs and conflict copies. Physical deletion would break reproducibility on the machine that uses
the asset, so the correct move is to cut the version-control link, not the file. An explicit
regeneration or fetch path keeps a fresh clone reproducible without carrying the bytes in history.

## Exceptions and boundaries

Assets whose exact bytes are required to reproduce a released artifact must remain reachable through
a documented channel before untracking; do not leave a build that silently depends on an untracked
file. Large-file storage extensions are an alternative when history matters, but migrating to them
is a coordinated change. Files under legal, licensing, or privacy constraints need an owner decision
rather than a bulk cleanup. Do not untrack during an unrelated change window.

## Example

A repository tracks high-resolution video renditions used only for local previewing, and the pack
size has grown to roughly the size of a small dependency tree. The renditions are untracked with the
cached-only removal so previews keep working, an ignore rule is added, and a short note in the
repository lists the local path and how to re-render the clips. The pack size drops, and a fresh
clone still builds because no build step read the videos.
