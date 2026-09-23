# Phase 13 — Generic HA Extraction and Reuse

This phase proves that HA is a reusable Unlayer infrastructure capability.

## Extraction

- [ ] Identify database-specific HA assumptions
- [ ] Remove database-specific logic from HA core
- [ ] Generic replicated-resource contract
- [ ] Generic node contract
- [ ] Generic membership contract
- [ ] Generic health contract
- [ ] Generic election contract
- [ ] Generic quorum contract
- [ ] Generic fencing contract
- [ ] Generic recovery contract
- [ ] Generic chaos hooks

## Database adapter

- [ ] Database replication adapter
- [ ] Database snapshot adapter
- [ ] Database recovery adapter
- [ ] Verify database behavior remains unchanged

## Reuse proof

- [ ] Fake non-database replicated resource
- [ ] Leader election against fake resource
- [ ] Quorum tests against fake resource
- [ ] Failover tests against fake resource
- [ ] Chaos tests against fake resource
- [ ] Verify HA core has no SQL knowledge

## Future services

- [ ] Identity integration contract
- [ ] Voice integration contract
- [ ] API/control-plane integration contract
- [ ] Resource-specific persistence contract

## Acceptance

- [ ] A second non-database service can use HA without copying database-specific implementation
