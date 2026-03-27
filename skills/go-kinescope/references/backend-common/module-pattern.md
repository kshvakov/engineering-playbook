---
description: Module pattern with interface return type and lowercase struct name
---

# Module pattern

## TL;DR

- Interface: `type Module interface { ... }` (not `ModuleInterface`).
- Implementation: `type module struct { ... }` (unexported).
- `New(...)` returns **`Module`**, not `*module`.
- Caching (ccache/memcached/redis) is **module-only** (never in handlers/service).

## Where it applies

- `internal/module/<name>/...` and call sites in `internal/service/...`.
