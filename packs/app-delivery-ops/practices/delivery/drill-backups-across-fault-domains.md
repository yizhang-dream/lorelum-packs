---
{"id":"app-delivery-ops.delivery.drill-backups-across-fault-domains","title":"Drill Backups Across Fault Domains","stage":"delivery","tech_stack":["web"],"applies_when":"a service stores user data on a server, and the team is about to rely on multi-device sync, disk redundancy, or an untested backup script as its protection","severity":"warn"}
---

## When to apply

Apply before declaring any data-bearing service production-ready, and whenever the only redundancy
is that clients hold copies. Also apply when the backup exists but has never been restored.

## Guidance

State the targets first: a recovery point objective (how much recent data may be lost) and a
recovery time objective (how long restore may take), written into a runbook. Then make the
mechanics match them:

- Write backups to a location in a separate fault domain from live data - a different volume or
  host, never inside the application data directory and never inside a build directory that a
  deploy replaces.
- Run the backup on a schedule (system cron or an equivalent scheduler), rotate old copies, and
  verify after each deploy that existing backups did not silently disappear.
- Add a restore drill to the automated test suite: pollute or delete data, detect the corruption,
  stop the spread, restore from backup, and assert the final state. Record the drill's pass count.
- Check the actual inventory of legacy backups before planning a migration; history described in
  documentation may already be gone from disk.

## Why

Replication by clients is not backup: a server-side merge bug propagates a bad state to every
client, so all copies are wrong at once. A backup stored inside the data directory or the build
output is destroyed by the same deploys and cleanups it is supposed to survive. An untested restore
is a hypothesis, not a capability - the failure is discovered exactly when data is already lost.

## Exceptions and boundaries

Fully regenerable data (caches, derived artifacts) may legitimately be excluded from backup; say so
explicitly instead of leaving a gap. Encryption of backups and off-site copies are desirable and
can be a later phase, but only as a written plan, never as an assumed property. A backup rotation
that has never been observed completing is not a backup either - alert on missed runs.

## Example

A sync server kept its only data in the application directory with no backups, protected in theory
by multi-device copies. The team added a daily archive job writing to a separate volume, put
recovery targets of one day and one hour into the runbook, and added a drill that corrupts a data
file, detects it, restores from the archive, and verifies the result. The drill passes in the
regular test run, so restore capability is demonstrated rather than assumed.
