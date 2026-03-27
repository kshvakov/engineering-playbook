---
description: Required imports and import organization for Kinescope Go projects
---

# Imports

## TL;DR

- Keep imports minimal: only import what the file actually uses.
- Grouping: standard library → external deps → internal Kinescope packages.
- Internal package paths must use `gitlab.kinescope.dev/...` (no “short local” path tricks).

## Common internal packages

```go
import (
    grpc_errors "gitlab.kinescope.dev/go/grpc/errors"
    "gitlab.kinescope.dev/go/render"
    "gitlab.kinescope.dev/go/validate"
    "gitlab.kinescope.dev/go/vault"
)
```
