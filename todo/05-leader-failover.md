# Phase 05 — Leader Election and Failover

## Election

- [ ] Election initiation
- [ ] Voting/quorum
- [ ] Term advancement
- [ ] Leader announcement
- [ ] Stale leader rejection
- [ ] Heartbeats
- [ ] Election timeout
- [ ] Fencing
- [ ] Split-brain prevention

## Failover

- [ ] Kill leader
- [ ] Detect failure
- [ ] Elect replacement
- [ ] Promote synchronized follower
- [ ] Resume writes
- [ ] Fence old leader
- [ ] Safely rejoin old leader
- [ ] Resynchronize old leader

## Partition tests

- [ ] Partition leader from one follower
- [ ] Partition leader from quorum
- [ ] Partition follower
- [ ] Heal partition
- [ ] Verify convergence
- [ ] Verify no split-brain writes

## Acceptance

- [ ] Repeated leader-failure tests produce one valid leader with no unexplained loss of committed quorum writes
