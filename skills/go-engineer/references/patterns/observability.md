## Observability — production essentials

### Logging
- Structured fields: `request_id`, `trace_id`, `operation`, `duration_ms`, `status` (avoid PII/secrets).
- Log errors **once**, typically at the boundary.
- Never log secrets/PII.

### Metrics
- Golden signals: latency, traffic, errors, saturation.
- Technical: goroutines, GC/heap, connection pools, queue depth.
- Domain: success signals for key operations.

### Tracing
- Spans around external calls (DB, HTTP, gRPC, queue).
- Context propagation is mandatory.

### Profiling (only if safe)
- pprof CPU/heap/block/mutex to validate hypotheses.
