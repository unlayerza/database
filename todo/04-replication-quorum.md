# Phase 04 — Replication and Quorum

## Replication

- [ ] Transaction replication format
- [ ] Database ID
- [ ] Sequence
- [ ] Term
- [ ] Event ID
- [ ] Integrity checksum
- [ ] Leader append
- [ ] Follower apply
- [ ] Durable acknowledgement
- [ ] Retry
- [ ] Duplicate handling
- [ ] Missing-range catch-up

## Quorum

- [ ] Replication factor
- [ ] Configurable write quorum
- [ ] Quorum success
- [ ] Quorum failure
- [ ] Continue with one failed follower when policy permits
- [ ] Stop when quorum is unavailable
- [ ] Expose quorum state

## Consistency

- [ ] Applied sequence
- [ ] Durable sequence
- [ ] Replication lag
- [ ] Read-after-write sequence
- [ ] Minimum-read-sequence enforcement

## Acceptance

- [ ] Successful writes have a demonstrable quorum durability guarantee
- [ ] Replicas converge to identical logical state
