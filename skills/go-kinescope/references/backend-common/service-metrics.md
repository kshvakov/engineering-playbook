---
description: Prometheus metrics handler with database pool metrics
---

# Service metrics

## TL;DR

- Prefer `promauto.*` for metrics (auto-registration).
- Export DB pool metrics: `max_open_connections`, `idle`, `in_use`.
- **High-cardinality labels are forbidden** (user_id, request_id, full URL, SQL, error message, payload).
- Allowed labels must be stable and low-cardinality (`status`, `method`, `route`, `grpc_method`, `result`).
- The Prometheus handler should update metric values before `promhttp.Handler().ServeHTTP(...)`.
