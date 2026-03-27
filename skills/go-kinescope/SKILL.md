---
name: go-kinescope
description: Kinescope conventions and patterns for Go services (HTTP/gRPC/common): project structure, errors, validation, render, proto, SQL, metrics, Makefile/spec.yml. Use together with go-engineer when working in gitlab.kinescope.dev style.
---

# Go Kinescope

This skill complements `go-engineer`: it focuses on **Kinescope-specific conventions** (service structure, standard packages, and “how it’s done here”), while `go-engineer` covers **general Go engineering** (correctness, reliability, security, observability, compatibility).

## How to use with `go-engineer`

- For architecture/risk/correctness/compatibility decisions: follow `go-engineer` and apply Kinescope conventions on top.
- For Kinescope project style (validation/render/errors/structure/Makefile/spec.yml): prioritize `go-kinescope`.
- If a convention conflicts with correctness/safety: choose the safer option and explain why.

## Quick start

1. Identify service type: HTTP / gRPC / common (worker).
2. Identify layer: transport boundary (handler/service) vs module (business logic).
3. Use standard Kinescope packages for validation/render/errors.
4. Keep SQL in `queries.go` and format it consistently.
5. Keep observability low-cardinality; implement correct shutdown.

## References

### Core (MUST/NEVER)
- [policy.md](references/backend-common/policy.md)
- [error-handling.md](references/backend-common/error-handling.md)

### Service templates
- [http-service.md](references/templates/http-service.md)
- [grpc-service.md](references/templates/grpc-service.md)
- [common-service.md](references/templates/common-service.md)

### Patterns (selected)
- [module-pattern.md](references/backend-common/module-pattern.md)
- [render.md](references/backend-common/render.md)
- [validation-http.md](references/backend-common/validation-http.md)
- [validation-grpc.md](references/backend-common/validation-grpc.md)
- [proto-pattern.md](references/backend-common/proto-pattern.md)
- [sql-formatting.md](references/backend-common/sql-formatting.md)
- [service-init.md](references/backend-common/service-init.md)
- [service-metrics.md](references/backend-common/service-metrics.md)
- [spec-yml.md](references/backend-common/spec-yml.md)
- [makefile.md](references/backend-common/makefile.md)
