---
description: Core architectural invariants and MUST/NEVER rules for Kinescope Go services
---

# Core Policy (MUST/NEVER)

## TL;DR

- **Context**: never use `context.Background()` in request paths; use `...Context(...)` methods.
- **Errors**: wrap with context, keep classifiable (`errors.Is/As`), log once.
- **Retries**: only at boundaries (handler/service/client), not inside modules.
- **Layers**: transport (gRPC/HTTP) → module (business logic). No business logic in transport.
- **Caching**: only in modules, never in handlers/service.
- **Standard packages**: `gitlab.kinescope.dev/go/render`, `gitlab.kinescope.dev/go/validate`, `gitlab.kinescope.dev/go/grpc/errors`.
- **Modules**: `New()` returns `Module` interface, not `*module`.

## Context & timeouts

- **NEVER** use `context.Background()` in handlers/service methods (request path).
- **ALWAYS** propagate `ctx` and respect deadlines/cancellation.
- Startup checks must use timeouts (`context.WithTimeout(context.Background(), ...)`).

## Errors

- Wrap errors with stable context (no PII / no dynamic payloads).
- Make decisions via `errors.Is/As` at boundaries.
- **LOG ONCE**: where you choose status/response or at top-level (worker loop / main).

## Retries

- Retries only at boundaries. Modules don’t do implicit retries.

## Layers

- Transport: decode/validate/format, translate errors.
- Module: business logic, DB, caching.
