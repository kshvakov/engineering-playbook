# Матрица `TaskType → RecommendedPrompts`

Используй это, когда:
- Router вернул `Other` (нужно предложить варианты), или
- пользователь указал `TaskTypeOverride` (нужно собрать `RecommendedPrompts`).

## Оглавление

- Матрица TaskType → пайплайн
- Примечания по нумерации

---

## Матрица TaskType → пайплайн

```text
Если TaskType = UX:
- 8.4 UX intake
- 1 Extract facts
- 2 Find gaps
- 3.1 Questions-to-author
- 4 Spec v1
- 5 AC + DoD
- 6 Corner cases
- 7 Test pack
- 8 Docs
- 9.1/9.2/9.3 Validators

Если TaskType = Perf:
- 8.1 Performance intake
- далее как выше

Если TaskType = Incident:
- 8.2 Incident intake
- затем (если есть evidence) 1→…→8 (опционально), обязательно 9.1 “без RCA без evidence”

Если TaskType = Migration:
- 8.3 Migration intake
- далее 1→…→8 (с фокусом на phases/reconciliation/rollback)

Если TaskType = SecurityAuthZ:
- 8.5 Security intake (AuthZ/RBAC)
- далее 1→…→8 (Docs и error model важны)

Если TaskType = SecurityKeys:
- 8.6 Security intake (Keys/Rotation)
- далее 1→…→8 (threat model lite, lifecycle, rollback)

Если TaskType = Integration:
- 0 Intake (опционально)
- 1→…→8 по базовому пайплайну
```

---

## Примечания по нумерации

- **Gate 0–6b**: core пайплайн (см. `gates-core.md`).
- **Docs** — это **Gate 7**, но промпт имеет ID `8) Docs` (исторически).
- **TaskType intake** — это `8.x` (см. `gates-tasktype.md`).
- **Validators** — `9.x` (см. `validators.md`).
