---
description: Error handling patterns for gRPC and HTTP services
---

# Error Handling

## TL;DR

- Wrap errors with `fmt.Errorf("...: %w", err)` or `grpc_errors.Wrap(err)`.
- Keep errors classifiable: `errors.Is/As`.
- Retries only at boundaries (handler/service/client), not in modules.
- Respect `context.Context` and use `...Context(...)` for DB/network operations.
- Log errors **once** (where handled/transformed or at top level).
- Use `errors.Join` for multiple errors.

## Rules

- Return errors as last parameter, never ignore.
- Wrap message answers “what were we doing” and is stable (no PII).
- Avoid `fmt.Errorf("%v", err)` — it breaks causality for `errors.Is/As`.
