# State-machine audit checklist

For each feature, examine:

## State representation
- [ ] Explicit enum or status values identified
- [ ] Boolean/nullable combinations enumerated
- [ ] Local, server, cache, persisted, and derived states distinguished
- [ ] Initial, pending, successful, failed, canceled, deleted, terminal states accounted for
- [ ] Illegal combinations specified as invariants

## Transitions and timing
- [ ] All triggers mapped (user, background, webhook, polling, recovery)
- [ ] Guards and permissions verified at server boundaries
- [ ] Duplicate, reordered, late, and lost events considered
- [ ] Concurrent actions, locking, serialization, and transactions reviewed
- [ ] Cancellation and disposal of outstanding work reviewed
- [ ] Timeout, retry/backoff, idempotency, and deduplication reviewed

## Persistence and sync
- [ ] Process death and app restart recovery
- [ ] Offline/online transition and queue resumption
- [ ] Local-remote version authority and conflict resolution
- [ ] Deletes, tombstones, orphans, partial writes, and reconciliation
- [ ] Multi-device and multi-account isolation
- [ ] DB constraint/RLS/function invariants match client assumptions

## Evidence
- [ ] Source locations recorded
- [ ] Expected vs actual transition documented
- [ ] Feasible reproduction or explicit uncertainty
- [ ] Existing test coverage inspected
- [ ] Severity separate from confidence
- [ ] Minimal regression test proposed
