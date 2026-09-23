# Phase 03 — Local Multi-Node Cluster

## Harness

- [ ] Node A on 7101
- [ ] Node B on 7102
- [ ] Node C on 7103
- [ ] Isolated storage per node
- [ ] Stable test identities
- [ ] Stop/restart/kill controls
- [ ] Cluster cleanup
- [ ] Timeout handling

## Cluster

- [ ] Membership discovery
- [ ] Initial leader election
- [ ] Cluster state inspection
- [ ] Leader write routing
- [ ] Node health inspection

## Tests

- [ ] Create database across required nodes
- [ ] Write and verify replicas
- [ ] Kill follower and continue
- [ ] Restart follower and catch up
- [ ] Kill leader and fail over
- [ ] Continue writes after failover
- [ ] Restart old leader and verify fencing
- [ ] Resynchronize old leader

## Acceptance

- [ ] Three real Bun processes survive automated leader failure
