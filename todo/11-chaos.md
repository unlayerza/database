# Phase 11 — Chaos Engineering

Chaos mode is mandatory production test infrastructure.

## Harness

- [ ] Chaos runner
- [ ] Deterministic seed
- [ ] Event recording
- [ ] Cluster-state recording
- [ ] Term recording
- [ ] Replication-position recording
- [ ] Reproducible failure reports

## Process chaos

- [ ] Random kill
- [ ] Random restart
- [ ] Leader kill
- [ ] Follower kill
- [ ] Multiple-node failure
- [ ] Full cluster restart

## Network chaos

- [ ] Delay
- [ ] Drop
- [ ] Duplicate
- [ ] Reorder
- [ ] Partition
- [ ] Partition healing

## Storage chaos

- [ ] Disk-full simulation
- [ ] Snapshot failure
- [ ] Backup failure
- [ ] Corrupt snapshot
- [ ] Corrupt history
- [ ] Stale local state

## Workload chaos

- [ ] Writes during failure
- [ ] Reads during failure
- [ ] High write concurrency
- [ ] High read concurrency
- [ ] Backup during failover
- [ ] Snapshot during load

## Assertions

- [ ] No split-brain writes
- [ ] No duplicate transaction effects
- [ ] No unexplained committed-write loss
- [ ] Followers converge
- [ ] Stale leaders are fenced
- [ ] Corrupt nodes are quarantined
- [ ] Recovery remains possible
- [ ] Backups remain independently usable

## Acceptance

- [ ] Seeded chaos runs are reproducible
- [ ] Repeated chaos campaigns preserve correctness invariants
