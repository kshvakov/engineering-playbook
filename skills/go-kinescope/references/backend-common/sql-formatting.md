---
description: SQL formatting rules (leading commas, AS aliases)
---

# SQL formatting

## Rules

- **Leading commas**: commas go **before** columns.
- **Aliases**: always use `AS` (`FROM table AS T`, not `FROM table T`).

## Example

```sql
SELECT
      M.media_id
    , P.type
    , P.share AS share
FROM workspace.entity AS E
JOIN media.media AS M ON M.media_id = E.media_id
JOIN workspace.privacy AS P ON P.privacy_id = M.privacy_id
WHERE E.entity_id = $1
    AND E.deleted_at IS NULL
```
