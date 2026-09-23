# Phase 01 — Storage Substrate

## Engine

- [ ] Integrate selected Turso Database/SQLite-compatible engine
- [ ] Create databases safely
- [ ] Open/close databases safely
- [ ] Configure required pragmas
- [ ] Verify WAL behavior
- [ ] Verify durability settings
- [ ] Verify commit/rollback
- [ ] Verify concurrent reads/writes
- [ ] Verify crash recovery
- [ ] Document engine-version assumptions

## Storage

- [ ] Database lifecycle abstraction
- [ ] Connection lifecycle
- [ ] Transaction abstraction
- [ ] Integrity checks
- [ ] Database metadata
- [ ] Size inspection
- [ ] WAL/storage inspection
- [ ] Safe snapshot primitive

## Tests

- [ ] Crash/reopen test
- [ ] Corruption detection test
- [ ] Consistent snapshot test
- [ ] Verify live-file copying is not used as backup primitive

## Acceptance

- [ ] Database survives tested process crashes with configured durability
- [ ] Snapshot can be opened and verified independently
