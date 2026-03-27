---
description: Proto file structure with validation tags, standard imports, and compilation
---

# Proto pattern

## TL;DR

- `protoc-gen-go-tags` is **mandatory** (generates `Validate()` and `*Form` structs).
- Import `kinescope/protobuf/tag.proto`.
- UUID: `bytes` + `(tag.uuid) = "kinescope"`.
- Validation: `(tag.validator) = "required,..."`, and in code use `req.Validate()` (not `validate.Struct()` on proto).
- The Makefile should use `go list -m` to locate the `protoc-gen-go-tags` module path.

## Minimal header

```protobuf
syntax = "proto3";

import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";
import "kinescope/protobuf/tag.proto";

option go_package = "github.com/kinescope/your-service/proto/servicename";
package servicename;
```
