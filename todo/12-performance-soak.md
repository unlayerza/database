# Phase 12 — Performance, Soak and Production Hardening

## Benchmarks

- [ ] Single-node reads
- [ ] Single-node writes
- [ ] Three-node local reads
- [ ] Three-node local writes
- [ ] Quorum latency
- [ ] Replication throughput
- [ ] Failover duration
- [ ] Recovery duration
- [ ] Snapshot duration
- [ ] PITR replay rate
- [ ] Backup throughput
- [ ] Memory per database
- [ ] Storage overhead

## Soak

- [ ] 1-hour soak
- [ ] 24-hour soak
- [ ] 72-hour soak
- [ ] 7-day target soak

## Resource safety

- [ ] File descriptor monitoring
- [ ] Memory leak detection
- [ ] Timer/task leak detection
- [ ] WAL growth monitoring
- [ ] Disk watermarks
- [ ] Replication backlog limits
- [ ] Backup backlog limits

## Operations

- [ ] Graceful shutdown
- [ ] Rolling restart
- [ ] Rolling upgrade
- [ ] Version compatibility
- [ ] Node drain
- [ ] Node replacement
- [ ] Emergency failover procedure
- [ ] Emergency restore procedure
- [ ] Backup verification procedure
- [ ] PITR procedure

## Acceptance

- [ ] No known critical resource leak remains
- [ ] Production-like failure tests pass
- [ ] Operational recovery procedures have been exercised
