# Phase 02 — Generic HA Core

## Node

- [ ] Stable node IDs
- [ ] Persisted node identity
- [ ] Duplicate identity detection
- [ ] Node address metadata
- [ ] Region/failure-domain metadata

## State machine

- [ ] Follower
- [ ] Candidate
- [ ] Leader
- [ ] Term/epoch
- [ ] State transitions
- [ ] Stale-term rejection
- [ ] Quorum calculation
- [ ] Membership model

## Replication

- [ ] Ordered events
- [ ] Positions
- [ ] Event IDs
- [ ] Idempotency
- [ ] Acknowledgement states
- [ ] Catch-up contract
- [ ] Snapshot bootstrap contract

## Health

- [ ] Heartbeats
- [ ] Health state
- [ ] Replication lag
- [ ] Failure detection

## Tests

- [ ] State transition tests
- [ ] Stale-term tests
- [ ] Quorum tests
- [ ] Duplicate event tests
- [ ] Missing-sequence tests

## Acceptance

- [ ] Generic HA can coordinate a fake replicated resource without database knowledge
