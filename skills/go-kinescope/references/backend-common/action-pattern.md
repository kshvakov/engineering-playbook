---
description: Action.go pattern for CLI flags, vault integration, and service initialization
---

# Action.go pattern

## TL;DR

- Use `vault.DSNFlag` for DSNs and `vault.StringFlag` for passwords/secrets.
- `vaultPassword` is set via LDFLAGS (see `makefile.md`); for local dev use `SKIP_DECRYPT=true`.
- Every flag must have `EnvVar`.
- Start the monitoring HTTP server in a goroutine via `mon.ListenAndServe(...)`.
- Return service init errors; use `log.Fatal` only if the monitoring server fails.
- Group flags by area (HTTP/GRPC/Postgres/Redis/Memcached).

## Invariants

- Build config in `action.Do()` and pass it to `service.New(cfg)`.
- Monitoring uses a dedicated address `MONITORING_HTTP_ADDR` (typically set by the deploy system in production).
