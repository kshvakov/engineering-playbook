---
description: File organization rules - ordering and splitting large files by {base}_{functionality}.go
---

# File organization

## Required order in a Go file

1. `package`
2. `import`
3. `const` (package-level, non-SQL)
4. `var` (package-level)
5. `init()` (if needed)
6. types (interfaces, structs)
7. functions/methods

## SQL placement

- SQL query constants must be in `queries.go` (not mixed into module/service/handler files).

## Splitting large files

- Keep base file (e.g. `service.go`, `module.go`, `handler.go`) with struct/interface + constructor + small helpers.
- Put grouped methods into `{base}_{functionality}.go` (examples: `service_route.go`, `module_list.go`, `handler_validation.go`).
