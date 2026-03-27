---
description: Naming conventions for packages, files, functions, variables, types, and interfaces
---

# Naming conventions

## TL;DR

- **Packages**: short, lowercase, descriptive (avoid `util`, `common`, `misc`).
- **Files**: one file = one main structure (`module.go`, `module_example.go`, `queries.go`).
- **Types**: PascalCase exported, camelCase unexported.
- **Functions**: PascalCase exported, camelCase unexported; constructors use `New()`.
- **SQL queries**: camelCase with `Query` suffix (`getExampleQuery`).
