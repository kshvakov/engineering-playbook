# Контракт override: как руками управлять `task-formalizer`

Цель: дать пользователю механизм “поправить” неверный роутинг и избежать “чёрного ящика”.

---

## 1) Override типа задачи (TaskType)

Добавьте в текст задачи строку:

```text
TaskTypeOverride: <UX|Perf|Incident|Migration|SecurityAuthZ|SecurityKeys|Integration>
```

Эффект:
- Router prompt не выполняется.
- `TaskType` фиксируется как override.
- `RecommendedPrompts` собирается по матрице из `tasktype-matrix.md`.

Когда использовать:
- Router вернул `Other`, но вы знаете тип.
- Router выбрал не тот тип.

---

## 2) Мини-override пайплайна (если нужен “не полный пакет”)

Если нужно ограничить объём, добавьте строку:

```text
FormalizerMode: <full|questions_only|routing_only>
```

Рекомендованные значения:
- `full`: полный пакет (по умолчанию).
- `questions_only`: только Routing + пробелы/вопросы (без Spec/AC/TestPack).
- `routing_only`: только Routing (для быстрой классификации).

Важно: если вы не хотите расширять интерфейс, этот override можно игнорировать (но тогда `task-formalizer` всегда full, кроме stop conditions).
