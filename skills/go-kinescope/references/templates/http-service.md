## HTTP service (Kinescope template)

Reference: “HTTP Service” template (Chi + `render` + `validate`) and strict layering handler/service/module.

### Expected layout

- `cmd/httpservice/main.go` — entrypoint, logging, `application_info`, CLI (`urfave/cli`)
- `cmd/httpservice/action/action.go` — flags+env, vault, build `config`, start monitoring server, `service.New()`/`Run()`
- `config/config.go` — typed config (`HTTP`, `Postgres`)
- `internal/service/` — orchestration, routing, lifecycle, shutdown
- `internal/module/` — business logic & DB (SQL in `queries.go`)
- `internal/handler/` — HTTP boundary: decode/validate → call module → `render.*`
- `internal/middleware/` — middleware (logging/metrics/correlation)
- `spec.yml`, `Makefile`, `version.go`, `.golangci.yml` (if used)

### Boundary rules (HTTP)

- **Validation**: only `gitlab.kinescope.dev/go/validate` (see `validation-http.md`)
- **Responses**: only `gitlab.kinescope.dev/go/render` (see `render.md`)
- **Errors**: wrap, keep classifiable, log once (see `policy.md`, `error-handling.md`)

### Checklist

- [ ] `main.go` sets up logrus and `application_info`, uses `urfave/cli`
- [ ] `action.go` defines flags with `EnvVar`, uses vault, starts monitoring via `mon.ListenAndServe` in a goroutine
- [ ] `service.New()` pings dependencies with timeouts, configures Postgres pool, implements correct shutdown
- [ ] handlers use `validate.Body()` → on error `render.JSONError()` → return
- [ ] modules contain business logic and DB; caching (if any) only here
- [ ] SQL in `queries.go`, formatted per `sql-formatting.md`
