## Code review checklist (Go, production-grade)

### Contracts & compatibility
- No silent contract break (gRPC/HTTP/events/schemas). If breaking — explicit migration plan.
- Error semantics/status codes are stable (or intentionally migrated).
- Config/flags changes include safe defaults and rollout strategy.
- New fields in API response are actually populated (not always null/empty).
- No duplicate JSON keys at different nesting levels.
- Struct field imports match actually used fields (no unused type imports for unpopulated fields).

### Context / timeouts / cancellation
- All long operations accept `context.Context` and respect cancel/deadlines.
- No unbounded retries/blocks; timeouts and limits exist.

### Errors
- Errors are wrapped with context (`%w`) but internal details do not leak externally.
- Boundary maps errors to external statuses/messages per target project `.cursor/rules`.
- Logs contain no secrets/PII.

### Concurrency
- No data races (maps, shared state, caches, metrics, tests).
- No goroutine leaks: lifecycle is owned (ctx/errgroup), clean shutdown exists.
- Backpressure/limits exist (worker pools, bounded queues); no unbounded buffers.

### Observability
- Correlation (request-id/trace-id) in logs.
- Metrics: latency + error rate + key resource/business signals.
- Tracing spans around external calls and critical sections.

### SQL / Database
- LATERAL/subquery JOINs: execute after pagination (OFFSET/LIMIT), not before. Aggregation on the full result set before pagination is a red flag.
- Total/count queries do not include JOINs needed only for data selection (not filtering).

### Branch hygiene
- All changes in the branch relate to the stated task. Unrelated fixes (log levels, feature flags, guard removals) belong in a separate commit or MR.
- Commit messages are descriptive (not `up`, `fix`, `wip`). Squash before MR if history is noisy.

### Tests
- Deterministic tests (no `sleep` as synchronization).
- Negative paths and edge cases covered.
- Use race detector for concurrent code when possible.
