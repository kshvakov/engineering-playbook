---
description: Version.go structure with build-time variables for version information
---

# version.go

## TL;DR

- The file must be at the **module root** (next to `go.mod`), not in `internal/` or subdirectories.
- The package name in `version.go` must match the module name (last path segment).
- Import it with a named import `info`:
  - `import info "github.com/kinescope/your-service"` or `import info "gitlab.kinescope.dev/your/your-service"`
- LDFLAGS use `module.VariableName` (no `/info` suffix).
- Variables:
  - via LDFLAGS: `ReleaseDate`, `AppVersion`, `GitCommit`, `GitBranch`
  - hardcoded: `Service`, `Namespace`

## Minimal template

```go
package servicename

var (
    ReleaseDate          = "now"
    AppVersion           = "v0"
    GitBranch, GitCommit = "branch", "commit"
    Service              = "service-name"
    Namespace            = "kinescope"
)
```
