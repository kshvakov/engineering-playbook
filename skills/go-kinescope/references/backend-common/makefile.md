---
description: Standard Makefile template with build, test, lint, proto, and deb targets
---

# Makefile pattern

## TL;DR

- Build env (prod): `GOOS=linux GOARCH=amd64 CGO_ENABLED=0`.
- Versioning via LDFLAGS: `ReleaseDate`, `GitCommit`, `GitBranch`, `AppVersion`.
- Vault: `vaultPassword` is also passed via LDFLAGS (see `action-pattern.md`).
- gRPC: `make proto` uses `protoc-gen-go-tags` and finds the module path via `go list -m` (not via `GOPATH`).

## Typical targets

- `help`, `build`, `clean`, `test`, `fmt`, `lint`, `deps`, `run`, `deb`
- for gRPC: `proto`
- `cursor-rules` (install dev rules if it’s the repo convention)
