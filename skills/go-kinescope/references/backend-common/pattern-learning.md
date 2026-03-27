---
description: Pattern learning and analysis system for continuous code quality improvement
---

# Pattern learning

Use only when the task is explicitly about “documenting a pattern / promoting a rule”.

## When to record a pattern

- The pattern appeared **2+ times** or is clearly becoming a standard.
- It impacts operational invariants: timeouts, retries, metrics/labels, shutdown order, degradation.

## Recording format (concise)

- Context (why it matters).
- Invariants (what must always be true).
- A small example snippet.
- When to apply / when not to apply.
