## Incident triage checklist (Go services)

### 0) Safety first
- Capture start time, impacted services/regions/clusters.
- Clarify impact: SLO breach, error rate/latency, data loss, financial risk.
- If risk is high: choose “stop the bleed” actions with minimal blast radius (rate limit, rollback, feature flag off, graceful degradation).

### 1) Define the symptom
- What exactly is failing? (errors, latency, timeouts, OOM, saturation)
- Scope: one endpoint/client/region or global?
- When did it start? Any deploy/config/migration around that time?

### 2) Fast hypotheses
- Deploy/config: regression, flag, migration
- Downstream: DB/queue/dependency issues → timeouts/deadlines
- Saturation: CPU/mem/conn pools, lock contention
- Data: bad data, cardinality explosion, bloat

### 3) Signals (cheap to expensive)
- Metrics: p95/p99 latency, error rate, saturation (CPU/mem/GC, goroutines), RPS, queue depth.
- Logs: error spikes, correlation via trace/request IDs.
- Traces: where time is spent, what times out.
- pprof (if safe): CPU/heap/block/mutex.

### 4) Low-risk actions
- Contain impact: rate limit, circuit breaker, disable heavy feature, rollback.
- Compare before/after (release, config, traffic).
- Find the “one big thing” dragging the system down (one endpoint/query).

### 5) Closure
- Root cause hypothesis + evidence.
- Fix + guardrails (limits, timeouts, alerts, tests).
- What to add to prevent recurrence (metrics/alerts/runbooks).
