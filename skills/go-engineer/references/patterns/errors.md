## Errors — production rules

### Goal
- Inside the system: errors must be **rich with context**.
- At the boundary (transport/boundary): expose **safe** messages and correct statuses/codes.

### Practice
- Add context while preserving the cause:
  - `fmt.Errorf("op: %w", err)`
- Don’t log the same error at every layer.
  Typically: log once at the boundary, below that return errors upward.
- Separate:
  - **internal** details (debug info, ids) vs
  - **external** response (safe message + status/code).

### Contracts
- Error mapping to external codes must follow the target project `.cursor/rules`.
  If rules are not available — stop and request them.

### Security
- Never include secrets/PII in error messages or logs.
  Use neutral identifiers (request-id, entity-id, shard) for debugging.
