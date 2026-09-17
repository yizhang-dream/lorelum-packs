---
{"id":"toolchain-deploy-ops.verification.triage-commit-exhaustion-freezes","title":"Triage Application Freezes as Commit Memory Exhaustion","stage":"verification","tech_stack":["windows","agent-ops"],"applies_when":"a desktop application freezes or restarts itself without a crash entry, heavy background work runs on the same machine, and the first suspicion is a bug in the application or in its graphics acceleration","severity":"warn"}
---

## When to apply

Apply when an interactive desktop application becomes unresponsive or restarts during heavy parallel
work, especially on a machine whose physical memory is smaller than the sum of the workloads. Also
apply before reporting a freeze as an application defect, a driver problem, or an out-of-memory crash
of the frozen process itself.

## Guidance

Start from the memory budget, not from the application. Check the system log for resource-exhaustion
events at the freeze timestamp, and check the application's own log for the absence of a crash entry:
a process killed by page thrash leaves no crash record, and the whole desktop stalls before anything
is terminated. Compare the commit limit with the commit charge - pagefile sizing caps how much memory
can be committed in total, independently of physical RAM.

Maintain a sampler: a scheduled task that runs every few minutes, records free commit and the largest
consumers by committed size, and raises an alert below a threshold. When a freeze happens, that log
is the only surviving snapshot of what was running. Read it for the pattern that identifies the
culprit class: a process with a large commit and zero working set is fully paged out - idle residue
or a leak - while a process of the same size with a live working set is genuinely active. Duplicate
processes competing for one listening port indicate leftovers from earlier runs.

Rank fixes by structure, not by symptom. Raising the pagefile lifts the ceiling but does not remove
the limit: the same schedule of concurrent heavy jobs will exhaust the larger budget too. The durable
fix is staggering - run one heavy batch at a time and stop the others (a rendering server, a sandbox
emulator, a long transcription job) before starting a competing load. Cap any container or virtual
machine subsystem in its configuration file, remembering that the cap applies only after that
subsystem is restarted and its services brought back up. Leaked service processes need administrative
removal, and expect them to return after a reboot.

## Why

At the moment of a freeze the visible symptom is always the interactive application, so attention goes
there while the real scarcity is a global commit budget consumed by idle and forgotten work. Graphics
acceleration is a common scapegoat, yet it is a victim: the driver is what stalls first when pages are
being swapped, not what consumed the memory. Only the sampler log separates the structural cause from
the last visible casualty.

## Exceptions and boundaries

Not every freeze is memory pressure: a single-process out-of-memory crash, a deadlock and a driver bug
look similar from the desktop. This check - resource-exhaustion events, no crash entry, commit charge
near the limit - is the discriminator. Do not disable hardware acceleration or reinstall graphics
drivers on this evidence. Raising the pagefile is legitimate first aid when it was previously reduced
below the system-managed size, but treat it as headroom, not as a fix.

## Example

A desktop application freezes twice in a week. The system log shows three resource-exhaustion events
inside a sixteen-minute window and the application log shows no crash entry; the commit budget, once
reduced to about 33 GB, had been raised to about 96 GB and was exhausted again. The sampler log lists
a GPU render server at 16.9 GB fully paged with zero working set, an active transcription job at
12.8 GB, a leaked service process at 11.1 GB, an uncapped container VM at 10.6 GB, and leftover
sandbox processes fighting over one port. The conclusion is a scheduling problem: cap the container
VM, remove the leaked service, and stagger the heavy jobs.
