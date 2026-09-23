# Unlayer Database — Architecture, Roadmap & Engineering Contract

> Foundational specification for the Unlayer managed database service.

This document is intentionally more detailed than a README. It is the engineering contract, architecture reference, roadmap, test strategy, recovery plan, and definition of done for Unlayer Database.

## 1. Mission

Unlayer Database is a managed SQLite-compatible database platform designed around:

- Bun-first infrastructure.
- One database per Unlayer organization by default.
- Replicated database nodes.
- Leader/follower operation.
- Quorum-based synchronous durability.
- Automatic failover.
- Incremental transaction-oriented replication.
- Safe snapshots and encrypted off-cluster backups.
- Point-in-time recovery (PITR).
- Automated node resynchronization.
- Local multi-process HA testing.
- Deterministic chaos testing.
- A reusable generic HA subsystem for future Unlayer services.

The service must optimize for correctness first, then performance, then operational convenience.

## 2. Core Principles

### Bun-first

Use Bun and web-standard primitives wherever practical. External dependencies require a technical justification. Native code may be introduced behind narrow interfaces when a subsystem genuinely benefits from it.

### SQLite is the storage authority

The database engine owns transaction, locking, WAL, recovery, and on-disk semantics. Unlayer must not invent a second transaction model.

SQLite WAL records committed changes separately from the main database and permits readers and writers to operate concurrently. The WAL is part of persistent database state; a live database cannot safely be treated as only its .db file. citeturn0search1turn0search0

### Replicate transactions/change history, not whole files

Normal replication must not copy an entire database file for every write. A committed transaction/change event is the unit of replication.

Whole-database snapshots are for bootstrap, backup, and recovery.

### No irreplaceable node

Every production database must have multiple durable copies according to its replication policy. Losing a node is an operational event, not a data-loss event.

### Replication is not backup

Replication provides availability and fast recovery. Backups provide recovery from accidental deletion, bad writes, corruption, ransomware, operator error, and total cluster loss.

### Control plane versus data plane

The control plane manages databases, nodes, clusters, placement, health, failover, backups, and recovery. The data plane performs database operations and replication. The control plane must not become a proxy for every SQL operation.

## 3. Target Architecture

```
                         UNLAYER CONTROL PLANE
                                  |
          +-----------------------+-----------------------+
          |                       |                       |
     DB REGISTRY             NODE REGISTRY          BACKUP REGISTRY
          |
          v
     DATABASE CLUSTER
          |
     +----+----+----+
     |         |    |
     v         v    v
   NODE A    NODE B NODE C
   LEADER    FOLLOWER FOLLOWER
     |         |    |
     +---------+----+
          replication
              |
              v
       snapshot/history
              |
              v
        Bunny Storage
```

A database is conceptually:

```
organization
    |
database
    |
cluster
    |
+---+---+---+
|   |   |   |
A   B   C
```

The organization-to-database relationship belongs to the Unlayer control plane. The database cluster owns the actual database state.

## 4. Database Lifecycle

### Create

1. Allocate database ID.
2. Register database.
3. Select cluster/placement.
4. Provision leader.
5. Initialize database.
6. Configure required settings.
7. Establish node identity.
8. Provision followers.
9. Verify synchronization.
10. Mark database available.

A database must not be advertised as HA-ready until its configured durability policy is satisfied.

### Normal operation

The initial target is:

```
A = leader
B = follower
C = follower
```

Writes enter through the leader. The leader assigns ordered replication positions and replicates committed transactions to followers.

### Delete

Deletion must distinguish logical deletion, cluster teardown, backup retention, and permanent purge. Retained recovery points must not disappear merely because the live database was deleted.

## 5. Replication

### Replication factor

Initial production target:

```
replication_factor = 3
```

The design must support larger factors without rewriting the protocol.

### Quorum

Initial target:

```
replication_factor = 3
write_quorum = 2
```

A healthy leader plus one healthy follower can therefore acknowledge a write even when the other follower is unavailable.

This is preferable to requiring every replica for every write because a single failed follower would otherwise become a write outage.

### Replication sequence

Every database has an ordered sequence:

```
100
101
102
103
...
```

A follower reports its highest applied/durable sequence. The leader can calculate lag and determine what history is missing.

### Event identity

Every replication event should identify:

- database ID,
- sequence,
- term/epoch,
- transaction/event ID,
- payload or change representation,
- checksum/integrity metadata.

### Idempotency

Delivery may be retried. Applying the same event twice must not duplicate its effect.

### Ordering

Events must be applied in order. If N+1 arrives before N, the follower must buffer/reject/request missing history rather than silently applying out of order.

### Acknowledgement semantics

A replication acknowledgement must distinguish at least:

- received,
- applied,
- durable.

A network response alone is not proof of durable storage.

## 6. Transaction Boundary

A multi-statement transaction is one atomic replication unit:

```
BEGIN
  statement 1
  statement 2
  statement 3
COMMIT
```

A rolled-back transaction produces no committed replication event.

Unlayer should build on the database engine's transaction boundary rather than replicating arbitrary SQL statements independently.

SQLite WAL itself uses a commit marker to establish transaction commit and supports multiple transactions in one WAL. citeturn0search1turn0search4

## 7. Leader, Followers and Failover

### Leader

The leader:

- accepts writes,
- serializes writes,
- assigns replication sequence,
- coordinates replication,
- tracks follower progress,
- participates in failover.

### Followers

Followers:

- maintain local database state,
- apply ordered history,
- report progress,
- may serve reads when consistency policy permits,
- may be promoted.

### Terms/epochs

Leadership must have a monotonically increasing term:

```
term 41 -> A leader
A fails
term 42 -> B leader
```

A stale node from an older term must not accept authoritative writes.

### Split-brain protection

The implementation must use explicit fencing and quorum-based election. A simple last-heartbeat-wins mechanism is not sufficient.

## 8. Read Architecture

Do not broadcast every read to every replica.

Initial policy may simply be:

```
writes -> leader
reads  -> leader
```

Follower reads can be introduced after replication correctness is proven.

Future read consistency modes may include:

- leader,
- local,
- at-least-sequence,
- strong.

A client that has written sequence 1042 must be able to request a read that is guaranteed not to come from a state older than 1042.

## 9. Generic HA Subsystem

Generic HA is a separate foundational repository: `unlayerza/ha`.

Database is the first stateful consumer and proving ground for that subsystem. The Database repository must not become the permanent owner of generic membership, election, quorum or fencing logic.

The leader/follower machinery must therefore be exposed through Database-specific adapters while generic cluster coordination remains in `unlayerza/ha`.

Target conceptual module:

```
unlayer/ha
  node identity
  membership
  health
  leader election
  terms/epochs
  quorum
  fencing
  replication positions
  failover
  recovery
  chaos hooks
```

Database-specific code should sit behind adapters:

```
Generic HA
  +-- Database
  +-- Identity
  +-- Voice
  +-- API/control plane
  +-- future services
```

The HA core must not know what SQL, SIP, identity records, or application objects mean.

Conceptual contract:

```
interface HANode {
  id: string
  address: string
  region?: string
}

interface ReplicationPosition {
  sequence: bigint
}

interface ReplicatedResource {
  append(event: Uint8Array): Promise<void>
  apply(event: Uint8Array): Promise<void>
  position(): Promise<ReplicationPosition>
  snapshot(): Promise<Uint8Array | AsyncIterable<Uint8Array>>
}
```

The exact interfaces may change during implementation.

## 10. Snapshots

A snapshot is a consistent database representation at a known replication position.

Metadata should include:

- database ID,
- snapshot ID,
- sequence,
- term,
- timestamp,
- engine version,
- schema version,
- size,
- checksum,
- encryption version.

Snapshots serve:

- follower bootstrap,
- node replacement,
- backup,
- PITR base points,
- disaster recovery.

Recovery can then be:

```
snapshot @ sequence 1000
+
history 1001..latest
=
current database
```

SQLite documentation explicitly warns that the WAL is part of database state and that copying the database file separately from its WAL can lose committed transactions or corrupt the result. Safe backup mechanisms include the backup API and VACUUM INTO. citeturn0search3turn0search1

## 11. Backup Strategy

Initial policy:

### Hourly

- Create a consistent recovery point every hour.
- Retain hourly recovery points for 48 hours.
- Expire hourly points after 48 hours.

### Daily

- Create a final daily recovery point.
- Retain daily points according to the service plan.

### Paid retention

Future plans may offer:

- extended hourly history,
- 7/30/90-day PITR,
- long-term daily retention,
- enterprise/custom retention.

Retention is a product policy; the underlying backup system must support configurable retention independently.

## 12. Bunny Storage

Bunny Storage is the off-cluster disaster-recovery destination, not the normal live read path.

Target:

```
database
  -> consistent snapshot
  -> compress
  -> encrypt
  -> checksum
  -> upload
  -> Bunny Storage
```

Live reads should not depend on object storage.

Backup metadata must include database ID, snapshot ID, sequence, timestamp, backup type, compression, encryption version, checksum, size, source node, engine version, and schema version.

Backups must be encrypted before leaving the trusted database environment. Encryption keys must not be stored beside encrypted backup objects.

A backup is not considered successful merely because upload returned success. Verification must prove the object exists, has the expected checksum, can be downloaded and decrypted, and can produce a valid database.

## 13. Point-in-Time Recovery

PITR is a major product feature and must be designed from the beginning.

Example:

```
snapshot @ 14:00
14:01 transaction 1001
14:02 transaction 1002
14:03 transaction 1003
...
14:37 transaction 1037
```

A request for 14:05 must reproduce the state at the corresponding transaction boundary.

Sequence numbers are authoritative for ordering. Timestamps are lookup metadata, not the sole ordering mechanism.

PITR should support:

- timestamp target,
- sequence target,
- latest valid point,
- oldest retained point.

By default, PITR must create a new recovery database rather than destructively overwrite production.

Before exposing a recovered database:

- verify snapshot checksum,
- verify history integrity,
- replay successfully,
- run database integrity checks,
- verify target sequence,
- record recovery metadata.

## 14. Node Bootstrap and Resynchronization

Joining node:

1. Authenticate.
2. Register identity.
3. Receive membership.
4. Select bootstrap snapshot.
5. Restore snapshot.
6. Record snapshot sequence.
7. Fetch missing history.
8. Apply in order.
9. Verify final state.
10. Become eligible for service.

If the node is only slightly behind, incremental catch-up is preferred. If it is too far behind, use a fresh snapshot plus remaining history.

A corrupt node must be quarantined and rebuilt rather than trusted.

## 15. Node States

Use explicit states:

```
provisioning
joining
snapshotting
catching_up
follower
leader
draining
failed
recovering
quarantined
retired
```

Transitions must be observable and testable.

## 16. Control Plane

The database control plane manages:

- database registry,
- node registry,
- cluster membership,
- placement,
- health,
- leader state,
- replication,
- backups,
- recovery jobs.

Conceptual API:

```
POST   /v1/databases
GET    /v1/databases
GET    /v1/databases/:id
DELETE /v1/databases/:id

GET /v1/databases/:id/cluster
GET /v1/databases/:id/health
GET /v1/databases/:id/replication

POST /v1/databases/:id/backups
GET  /v1/databases/:id/backups
POST /v1/databases/:id/backups/:backupId/verify

POST /v1/databases/:id/restore
POST /v1/databases/:id/pitr
```

The public SQL/data API should remain distinct from administrative control-plane operations.

## 17. Multi-Tenancy

Default mapping:

```
organization
  -> database_id
  -> cluster
  -> nodes
```

All access must be authorized against control-plane state. A client-supplied database ID is never sufficient authorization.

Cross-organization access must be impossible by default.

## 18. Resource Isolation

Track and enforce:

- database size,
- database count per node,
- connection limits,
- concurrent operations,
- backup bandwidth,
- replication bandwidth,
- disk watermarks,
- CPU/memory pressure.

A noisy database must not destabilize unrelated databases.

## 19. Security

Required:

- authenticated node-to-node traffic,
- authenticated control-plane traffic,
- per-node identity,
- authorization for cluster operations,
- encryption in transit,
- encrypted backups,
- secret rotation,
- replay protection,
- audit logging,
- rate limits,
- resource limits,
- tenant isolation.

A compromised node must not automatically obtain unrestricted control over every database.

## 20. Auditability

Record privileged operations including:

- database creation/deletion,
- backup creation/deletion,
- restore,
- PITR,
- failover,
- node quarantine,
- membership changes.

Audit records should contain actor, target, operation, timestamp, result, request ID, and relevant metadata.

## 21. Testing Strategy

Testing is part of the architecture.

Required levels:

1. Unit tests.
2. Integration tests.
3. Real multi-process cluster tests.
4. Failure-injection tests.
5. Chaos tests.
6. Backup/restore tests.
7. PITR tests.
8. Performance tests.
9. Soak tests.

The important HA tests must exercise actual Bun server processes, not only mocks.

## 22. Local HA Harness

Local development must support:

```
node-a -> 127.0.0.1:7101
node-b -> 127.0.0.1:7102
node-c -> 127.0.0.1:7103
```

Each node receives independent storage.

The harness must support:

- start,
- stop,
- hard kill,
- restart,
- message delay,
- message drop,
- message duplication,
- message reordering,
- partition,
- storage corruption simulation,
- cluster inspection,
- replication inspection,
- deterministic cleanup.

This is intended to approximate production behavior on a single developer machine. Internet latency and physical failure domains must still be tested separately.

## 23. Chaos Mode

Chaos mode is a major feature, not an optional test script.

Inject:

- random node termination,
- leader termination,
- follower termination,
- multiple-node failure,
- delayed replication,
- dropped messages,
- duplicated messages,
- reordered messages,
- network partitions,
- partition healing,
- disk-full conditions,
- backup failures,
- restore failures,
- stale-node rejoin,
- slow nodes,
- high write load,
- high read load,
- snapshot during writes,
- backup during failover.

Chaos must support deterministic seeds.

Example:

```
CHAOS_SEED=12345
```

Every failed campaign must report the seed, injected events, cluster state, terms, sequences, and final state.

## 24. Mandatory HA Scenarios

### Basic replication

Create -> write -> verify replicas.

### Leader failure

Write -> kill leader -> elect follower -> write again -> verify convergence.

### Follower failure

Kill follower -> continue writes -> restart -> catch up -> verify.

### Duplicate event

Deliver the same replication event twice -> verify one logical effect.

### Out-of-order event

Deliver N+1 before N -> verify buffering/rejection -> deliver N -> verify convergence.

### Partition

Partition nodes -> verify quorum behavior and no split brain -> heal -> verify convergence.

### Old leader return

Leader A -> A fails -> B becomes leader -> writes continue -> A returns -> A is fenced -> A resynchronizes.

### Full restart

Stop every node -> restart cluster -> recover -> elect leader -> verify data.

## 25. Mandatory PITR Scenarios

Create:

```
snapshot @ 100
write A -> 101
write B -> 102
write C -> 103
write D -> 104
```

Restore at 102 and verify A/B exist while C/D do not.

Also test:

- exact transaction boundary,
- latest point,
- oldest retained point,
- expired point,
- missing history,
- corrupt history,
- corrupt snapshot,
- recovery into a new database,
- recovery followed by new writes.

## 26. Backup Tests

Test:

- snapshot creation,
- checksum,
- compression,
- encryption,
- upload,
- remote verification,
- download,
- decryption,
- restore,
- retention,
- hourly expiration,
- daily retention,
- failed upload,
- interrupted upload,
- corrupt object,
- missing object.

Periodically perform an actual restore test from the remote backup location.

## 27. Performance and Soak

Measure:

- read latency,
- write latency,
- quorum latency,
- replication latency,
- failover duration,
- recovery duration,
- snapshot duration,
- PITR replay rate,
- backup throughput,
- memory per database,
- storage overhead.

Compare:

- single node,
- three-node local cluster,
- real network cluster,
- degraded cluster.

Run long-lived tests to expose memory leaks, WAL growth, replication backlog, file descriptor leaks, stale membership, and resource exhaustion.

## 28. Observability

Structured events should include:

- node_started,
- node_joined,
- node_left,
- leader_elected,
- leader_lost,
- term_changed,
- replication_started,
- replication_applied,
- replication_failed,
- quorum_reached,
- quorum_lost,
- snapshot_started,
- snapshot_completed,
- backup_started,
- backup_completed,
- backup_failed,
- restore_started,
- restore_completed,
- pitr_started,
- pitr_completed,
- node_quarantined,
- node_recovered.

Metrics should include replication lag, write/read latency, quorum latency, leader changes, failed replications, snapshot duration, backup age, recovery duration, database size, WAL size, and disk usage.

## 29. Correctness Invariants

These are non-negotiable:

1. At most one valid leader may accept authoritative writes for a term.
2. Committed replication events have strict ordering.
3. Replication application is idempotent.
4. Successful writes satisfy the configured quorum durability policy.
5. Stale leaders cannot overwrite newer leadership state.
6. Recovered nodes converge to authoritative state.
7. Retained backups survive live-cluster loss.
8. PITR is deterministic for the same snapshot/history/target.
9. Tenants cannot access one another's databases.
10. No committed state is silently discarded.

## 30. Failure Model

Expected failures:

- process crash,
- server reboot,
- node loss,
- network interruption,
- network partition,
- follower lag,
- leader failure,
- disk pressure,
- backup failure,
- stale node,
- partial deployment.

Eventually supported:

- regional node loss,
- entire cluster loss,
- corrupted node,
- corrupted backup,
- accidental destructive writes.

Not automatically solved:

- compromise of the control plane,
- compromised credentials,
- destruction of every backup location.

Those require separate security and disaster-recovery controls.

## 31. Production Readiness

Production readiness requires more than CRUD.

The minimum meaningful acceptance lifecycle is:

```
create database
write significant dataset
replicate
kill leader
fail over
continue writes
partition nodes
heal partition
restart failed nodes
resynchronize
take backup
destroy cluster
restore from backup
perform PITR
verify historical state
resume writes
```

This lifecycle must pass repeatedly.

## 32. Non-Goals

The first version is not intended to become:

- a general distributed SQL engine,
- a replacement for SQLite,
- a generic scheduler,
- an object-storage system,
- a full observability vendor,
- a globally distributed transaction system.

## 33. Open Engineering Questions

These must be resolved through current documentation and experiments rather than assumptions:

- Exact replication primitives exposed by the selected Turso Database version.
- Exact transaction/change representation.
- Exact leader-election protocol.
- Exact quorum semantics.
- Cross-region quorum policy.
- Snapshot implementation available through current bindings.
- Backup encryption format and key lifecycle.
- Bunny Storage API/limits.
- Maximum practical databases per node.
- Maximum replication factor.
- Follower-read consistency implementation.
- Schema migration strategy.
- Replication-history retention.
- History compaction.
- Snapshot/history checksum chaining.
- Corrupt-node detection and quarantine.
- Engine-version migration strategy.

## 34. Development Order

1. Storage substrate.
2. Integrate the external `unlayerza/ha` substrate.
3. Local multi-node harness.
4. Replication positions/events.
5. Database transaction integration.
6. Quorum.
7. Leader election.
8. Failover.
9. Resynchronization.
10. Snapshots.
11. Backups.
12. PITR.
13. Control plane.
14. Public API.
15. Security/isolation.
16. Chaos.
17. Performance/soak.
18. Production hardening.
19. Cross-service HA reuse proof.

## 35. First Milestone

The first real milestone is:

```
3 Bun server processes
3 independent local database directories
1 database
1 leader
2 followers
quorum write
leader failure
new leader
continued writes
old leader returns
automatic resynchronization
```

Then add chaos, snapshots, PITR, and finally the cloud control plane.

## 36. Architectural Rule for Future Services

Identity, Voice, API, Hosting, and future infrastructure services should consume the generic HA subsystem rather than reimplementing leader/follower behavior.

The database repository is therefore:

1. the first stateful consumer of Unlayer HA, and
2. the proving ground for the Database-specific HA adapter.

Generic HA ownership lives in `unlayerza/ha`. Identity and Voice should adopt the same HA contracts without importing Database-specific behavior.

---

### Reference material

Consult current official documentation before relying on engine-specific behavior:

- SQLite WAL: https://www.sqlite.org/wal.html
- SQLite WAL format/recovery: https://www.sqlite.org/walformat.html
- SQLite corruption/backup guidance: https://www.sqlite.org/howtocorrupt.html
- SQLite file format: https://www.sqlite.org/fileformat.html

Engine-specific Turso behavior must be verified against the exact version selected by the project.
