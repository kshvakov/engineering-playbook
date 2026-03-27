## Common service (Kinescope template)

Reference: “Common Service” template for workers/background tasks (no HTTP/gRPC), with correct lifecycle and explicit degradation for optional dependencies.

### Expected layout

- `cmd/commonservice/main.go` — entrypoint, logging, `application_info`, CLI (`urfave/cli`)
- `cmd/commonservice/action/action.go` — flags+env, vault, build `config`, monitoring server, `service.New()`/`Run()`
- `config/config.go` — typed config (Postgres + optional Redis/Memcached/ClickHouse + shutdown timeout)
- `internal/service/` — orchestrates background work and graceful shutdown
- `internal/module/` — business logic & DB (SQL in `queries.go`)
- `spec.yml`, `Makefile`, `version.go`

### Dependency rules

- **Postgres**: critical — fail-fast on startup (ping with timeout), pool configured (see `service-init.md`)
- **Redis/ClickHouse**: may degrade only by explicit contract (warn + continue), not silently (see `policy.md`, `service-init.md`)
- **Shutdown**: always time-bounded; closing resources must not hang forever

### Checklist

- [ ] `main.go` sets up logrus and `application_info`, uses `urfave/cli`
- [ ] `action.go` defines flags with `EnvVar`, uses vault, starts monitoring
- [ ] `service.New()` pings deps with timeouts; degradation only for explicitly optional deps
- [ ] `Run()` waits for signals, stops workers, closes connections within timeout
- [ ] SQL in `queries.go`, formatted per `sql-formatting.md`
