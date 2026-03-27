---
description: Code style patterns for improved readability and visual perception
---

# Code style

## TL;DR

- **Local vars**: for 3+ related variables, group them in a `var (...)` block (dependencies first, then visual compactness).
- **Struct literals**: field order in literals should match the order in the struct definition (single source of truth).
- **Inline simple computations**: inline one-off simple scalars at the use site (if it doesn’t hurt readability).

## Constraints

- Avoid “style-only” diffs without a reason; apply when you’re already touching nearby code.
