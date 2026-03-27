---
description: Validation patterns for configuration structs using gitlab.kinescope.dev/go/validate
---

# Validation (config)

## TL;DR

- For config structs use `validate.Struct(cfg)`.
- Return validation errors as `validate.ErrInvalidParams(...)` and wrap with context.
- Validate at initialization (boundary), not inside business logic.
