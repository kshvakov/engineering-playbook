---
description: Configuration structures for gRPC and HTTP services
---

# Config pattern

## TL;DR

- Group fields into sub-structs: `Postgres`, `HTTP`, `Redis`, `Memcached`, `Shutdown`.
- All timeouts/lifetimes are `time.Duration`.
- Field names reflect env var names (without prefixes); defaults live in `action.go` flags.
- Config structs contain **no** validation/logic; validate/ping/degrade in `service.New()` (see `service-init.md`).
- Sensitive values (DSN/Password) in production are handled via vault (see `action-pattern.md`).
