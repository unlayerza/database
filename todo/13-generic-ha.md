# Phase 13 — Generic HA Extraction and Reuse

> Generic HA is now a separate repository: `unlayerza/ha`. This phase is the Database-side migration/proof phase, not a plan to keep HA implementation inside Database.

## Database responsibilities

- [ ] Identify every Database-specific HA assumption
- [ ] Define the Database adapter for `unlayerza/ha`
- [ ] Map Database leader/authority state
- [ ] Map replication positions
- [ ] Map Database quorum semantics
- [ ] Map Database fencing requirements
- [ ] Map Database recovery/resynchronization hooks
- [ ] Verify Database behavior remains unchanged

## Removal

- [ ] Remove generic node/membership implementation from Database
- [ ] Remove generic election implementation from Database
- [ ] Remove generic quorum implementation from Database
- [ ] Remove generic fencing implementation from Database
- [ ] Remove generic lifecycle/chaos implementation from Database
- [ ] Keep only Database-specific replication and recovery logic

## Reuse proof

- [ ] Database consumes `unlayer/ha` without copying its implementation
- [ ] Three-node Database cluster uses HA contracts
- [ ] Leader failure test passes
- [ ] Stale leader is fenced
- [ ] Node recovery/resynchronization passes
- [ ] Database-specific tests remain green

## Cross-service proof

- [ ] Identity can consume the same HA contracts
- [ ] Voice can consume the same HA contracts
- [ ] HA core contains no SQL/SIP/identity knowledge

## Acceptance

- [ ] `unlayerza/ha` is the single generic HA implementation
- [ ] Database is an HA consumer
- [ ] No second generic HA implementation remains in Database
