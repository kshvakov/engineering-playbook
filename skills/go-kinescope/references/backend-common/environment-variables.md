---
description: Environment variables configuration for services, databases, Redis, Memcached
---

# Environment variables

## TL;DR

- Env var names use `UPPER_SNAKE_CASE`.
- Group by prefixes: `POSTGRES_*`, `REDIS_*`, `MEMCACHED_*`, `HTTP_*`, `GRPC_*`.
- Suffixes: `_ADDR`, `_DSN`, `_PASSWORD`, `_TIMEOUT`, `_CONNS`.
- `MONITORING_HTTP_ADDR` is typically set automatically in production (don’t hardcode it in `spec.yml`).
- For vault: use `SKIP_DECRYPT=true` for local dev (see `action-pattern.md`).

## Typical variables

- Common: `DEBUG`, `SKIP_DECRYPT`, `MONITORING_HTTP_ADDR`
- HTTP: `HTTP_ADDR`, `HTTP_READ_TIMEOUT`, `HTTP_WRITE_TIMEOUT`, `HTTP_IDLE_TIMEOUT`, `HTTP_SHUTDOWN_TIMEOUT`
- gRPC: `GRPC_ADDR`
- Postgres: `POSTGRES_DSN`, `POSTGRES_MAX_IDLE_CONNS`, `POSTGRES_MAX_OPEN_CONNS`, `POSTGRES_CONN_MAX_LIFETIME`, `POSTGRES_CONN_MAX_IDLE_TIME`
- Redis (if needed): `REDIS_ADDRS`, `REDIS_MASTER_NAME`, `REDIS_USERNAME`, `REDIS_PASSWORD`, `REDIS_DB`, `REDIS_DIAL_TIMEOUT`, `REDIS_READ_TIMEOUT`, `REDIS_WRITE_TIMEOUT`
- Memcached (if needed): `MEMCACHED_ADDRS`, `MEMCACHED_DIAL_TIMEOUT`, `MEMCACHED_CONN_MAX_LIFETIME`, `MEMCACHED_MAX_IDLE_CONNS_PER_ADDR`
