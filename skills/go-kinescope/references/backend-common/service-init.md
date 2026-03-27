---
description: Service initialization pattern with database connections and graceful shutdown
---

# Service init

## TL;DR

- On startup, ping external dependencies with **timeouts**.
  - critical deps: fail-fast
  - cache/analytics: degrade only by explicit contract
- Postgres:
  - `sqlx.Open("lego-go", dsn)`
  - always configure pool: `SetMaxIdleConns`, `SetMaxOpenConns`, `SetConnMaxLifetime`, `SetConnMaxIdleTime`
  - `PingContext` with timeout
- Shutdown:
  - HTTP: `server.Shutdown(ctx)` with timeout
  - gRPC: `GracefulStop()` → on timeout `Stop()`
- Modules receive `*sqlx.DB`, and `New()` returns the `Module` interface.
