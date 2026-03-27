---
description: Modern Go 1.21+ practices and features
---

# Modern Go (1.21+)

## TL;DR

- Use `strings.Cut` instead of `strings.Split` for parsing “key:value”.
- Use `slices.*` / `maps.Clone` for common operations.
- Use `errors.Is/As/Join` for errors.
- Prefer `any` over `interface{}`.

## Diff rule

- Avoid “modernize-only” diffs without a reason; modernize when you’re already touching nearby code.
