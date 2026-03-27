---
description: Validation patterns for gRPC services using gitlab.kinescope.dev/go/validate
---

# Validation (gRPC)

## TL;DR

- For proto requests (with `protoc-gen-go-tags`) use `req.Validate()`.
- Wrap request/validation errors at the boundary: `grpc_errors.Wrap(err)`.
- For non-proto structs (internal DTOs/configs): `validate.Struct(req)` → `validate.ErrInvalidParams(...)` → wrap.
