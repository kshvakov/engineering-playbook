---
description: HTTP response rendering using gitlab.kinescope.dev/go/render package
---

# Render (HTTP)

## TL;DR

- Use only `gitlab.kinescope.dev/go/render` (do not create custom render packages).
- Success:
  - `render.JSON(w, data[, meta])` — responses with data
  - `render.JSONSuccess(w)` — success without data
  - `render.JSONRaw(w, any)` — custom structure without `{data/meta}` wrapper
  - `render.JSONStream(w)` — NDJSON/JSONL streaming
- Errors: always `render.JSONError(w, err)` (including validation errors).

## Notes

- The package sets headers automatically (`Content-Type`, `X-Request-ID`).
- It maps many standard errors (SQL, Redis, Memcached, gRPC status, JSON, time parse).
