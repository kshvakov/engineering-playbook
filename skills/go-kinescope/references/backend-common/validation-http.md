---
description: Validation patterns for HTTP handlers using gitlab.kinescope.dev/go/validate
---

# Validation (HTTP)

## TL;DR

- Decode + validate body: `validate.Body(r.Body, &req)`.
- Return the error via the standard path: `render.JSONError(w, err)` and `return`.
- Do not write custom validators/renderers — use the standard packages.
