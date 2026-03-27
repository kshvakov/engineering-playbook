# Формат вывода `task-formalizer` (Output Pack + прозрачность)

Этот файл фиксирует **строгую структуру** результата, чтобы `task-formalizer` не был “чёрным ящиком”.

---

## 1) Routing block (обязателен всегда)

```text
## Routing
TaskType: <UX|Perf|Incident|Migration|SecurityAuthZ|SecurityKeys|Integration|Other>
MustHaveInputs:
- ...
RecommendedPrompts:
- ...
StopConditions:
- ...
RoutingDecision:
- <коротко почему выбран тип; если нельзя обосновать без домыслов — написать UNKNOWN>
```

---

## 1.1) Task summary (rewritten) (обязателен всегда)

Сразу после `## Routing` добавь секцию **переписанного резюме задачи**, чтобы сырой текст стал читаемым.

Требования:
- **только по фактам входа** (никаких “как обычно”, никаких достроек);
- если чего-то нет во входе — это не “дописывается”, а уходит в `Unknowns`/вопросы;
- цель: “понятная постановка в 30 секунд”, которую можно вставить в тикет.

Шаблон (строго):

```text
## Task summary (rewritten)
### Проблема / контекст
...

### Цель
...

### Что меняем (scope)
...

### Критерии успеха
...

### Требуется уточнить
- ...
```

---

## 2) Stop-conditions behavior (если сработали блокеры)

Если есть блокеры из `MustHaveInputs` и/или выполнено любое `StopConditions`, то:
- **не генерировать** Spec/AC/TestPack “как будто всё известно”;
- вернуть только:
  - `## Routing`
  - `## Task summary (rewritten)`
  - `## P0 questions (blockers)` (или `## Unknowns (P0 сверху)`)
  - `## What to provide next` (список требуемых входов)
  - `## ExecutionTrace`

---

## 3) Output pack (полный режим)

```text
## Routing
...

## Task summary (rewritten)
...

## Intake (Gate 0)
<структурированный конспект или ссылка на секции ниже>

## Facts / Entities / Invariants (Gate 1)
## Gaps / Contradictions / Risks (Gate 2)
## Questions (Gate 3) / Questions-to-author (Gate 3.1)
## Spec v1 (Gate 4)
## Acceptance Criteria + DoD (Gate 5)
## Corner cases (Gate 6a)
## Test pack (Gate 6b)
## Docs notes (Gate 7)

## Validators
### 9.1 Hallucinations / Unsupported claims
### 9.2 Verifiability
### 9.3 Ambiguity

## Assumptions (если есть)
## Open questions (если остались)

## Next steps
- ...

## ExecutionTrace
Applied:
- <номер промпта/гейта> — <что получили>
Skipped:
- <номер> — <почему (stop condition / недостаточно входа / не применимо)>
```

---

## 4) Поведение при `TaskType=Other` (и как исправлять)

Если Router вернул `Other`:
- `RoutingDecision` должен перечислить 2–4 кандидата (`Integration`/`Migration`/`Security*`/...) и что мешает выбрать один.
- `RecommendedPrompts` должен включать минимум:
  - `0 Intake`, `1 Extract facts`, `2 Find gaps`, `3.1 Questions-to-author`
- В `Next steps` дать инструкцию:
  - “Если тип понятен — укажи `TaskTypeOverride: <...>` и перезапусти”.
