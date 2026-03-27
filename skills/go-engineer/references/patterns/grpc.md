## gRPC — production reminders

### Deadlines / cancellation
- Servers must respect client deadlines.
  At the boundary: apply safe defaults if the client provides none.

### Statuses and errors
- External contract: stable `codes.*` and safe messages.
  Mapping must follow the target project `.cursor/rules`.

### Interceptors (typical order)
- request-id / correlation
- authn/authz
- rate limiting
- tracing
- logging
- recovery
- metrics

### Retries / idempotency
- Retry only idempotent operations.
  Otherwise use idempotency keys / deduplication.

### Streaming
- Enforce backpressure and limits (messages, bytes, time).
- Always respect ctx and close resources.
