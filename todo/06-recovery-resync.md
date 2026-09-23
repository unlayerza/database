# Phase 06 — Recovery and Resynchronization

## Bootstrap

- [ ] Authenticate joining node
- [ ] Register node
- [ ] Assign database membership
- [ ] Select snapshot
- [ ] Transfer snapshot
- [ ] Verify checksum
- [ ] Restore snapshot
- [ ] Record sequence
- [ ] Replay missing history
- [ ] Verify final state
- [ ] Mark caught up

## Incremental catch-up

- [ ] Detect lag
- [ ] Determine missing range
- [ ] Transfer range
- [ ] Apply range
- [ ] Verify sequence continuity
- [ ] Verify state

## Full rebuild

- [ ] Detect unrecoverable local state
- [ ] Quarantine node
- [ ] Reinitialize safely
- [ ] Bootstrap snapshot
- [ ] Replay history
- [ ] Verify integrity
- [ ] Rejoin

## Acceptance

- [ ] A failed node can be destroyed, recreated and resynchronized without manual database surgery
