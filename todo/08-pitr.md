# Phase 08 — Point-in-Time Recovery

## History

- [ ] Ordered transaction history
- [ ] Sequence metadata
- [ ] Term metadata
- [ ] Timestamp metadata
- [ ] History retention
- [ ] History compaction
- [ ] Snapshot-to-history relationship

## Recovery

- [ ] Restore latest snapshot
- [ ] Select snapshot before target
- [ ] Replay history
- [ ] Stop at exact target
- [ ] Timestamp target
- [ ] Sequence target
- [ ] Separate recovery database
- [ ] Validate recovered database
- [ ] Record recovery metadata

## Safety

- [ ] Reject expired point
- [ ] Detect missing history
- [ ] Detect corrupt history
- [ ] Detect corrupt snapshot
- [ ] Never overwrite production by default
- [ ] Explicit promotion/switchover

## Tests

- [ ] Exact transaction
- [ ] Before destructive transaction
- [ ] After destructive transaction
- [ ] Latest point
- [ ] Oldest retained point
- [ ] Expired point
- [ ] Missing history
- [ ] Corrupt history
- [ ] Corrupt snapshot
- [ ] Restore and resume writes

## Acceptance

- [ ] Known historical state can be deterministically recreated
