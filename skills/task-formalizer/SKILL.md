---
name: task-formalizer
description: Формализация сырых задач в проверяемый «пакет артефактов» для разработки/QA (Spec v1/PRD-lite, вопросы P0–P2, пробелы/противоречия, AC/DoD, тест‑матрица/тест‑план, docs notes) с жёстким запретом на домыслы. Использовать, когда нужно быстро превратить неформализованную задачу в структуру, выявить неизвестные и подготовить материал для планирования/реализации/тестирования. Всегда показывает прозрачный роутинг (TaskType/MustHaveInputs/RecommendedPrompts/StopConditions) и поддерживает TaskTypeOverride.
---

# Task Formalizer (RU)

Цель: превращать любую “сырую” задачу в **формализованный, проверяемый пакет** для разработки/QA, не выдумывая фактов.

Скилл **не чёрный ящик**:
- в начале всегда показывает `TaskType` и почему он выбран;
- в конце показывает `ExecutionTrace` (что реально запускалось и что пропущено).

## Быстрый старт (как пользоваться)

1) Вставь текст задачи (как есть).
2) Если роутер ошибся или вернул `Other` — укажи override строкой и повтори:

```text
TaskTypeOverride: <UX|Perf|Incident|Migration|SecurityAuthZ|SecurityKeys|Integration>
```

3) Скопируй нужные секции из результата в таск/доки. По умолчанию скилл возвращает **один** “пакет”, а не россыпь документов.

## Библиотека (что читать по мере надобности)

- `references/router-prompt.md` — router prompt (только маршрутизация и блокеры).
- `references/gates-core.md` — core гейты 0–8 (включая Docs/Gate 7).
- `references/gates-tasktype.md` — спец‑интейки 8.x (Perf/Incident/Migration/UX/Security*).
- `references/validators.md` — валидаторы 9.1–9.3 (ревью результата).
- `references/tasktype-matrix.md` — матрица `TaskType → RecommendedPrompts` (для `TaskTypeOverride` и случаев `Other`).
- `references/artifacts-templates.md` — шаблоны Spec/AC/DoD/Test/Docs.
- `references/output-pack-format.md` — строгий формат результата, stop-conditions, `ExecutionTrace`.
- `references/quality-rubric.md` — чек‑листы качества.
- `references/override-contract.md` — контракт override.
- `real-cases.md` — реальные кейсы и переиспользуемые промпты (копилка для эволюции playbook).

## Карта процесса (в 10 секунд)

`TaskType` → `## Routing` → (StopConditions?) → Core gates → TaskType intake (если нужно) → Validators → `## ExecutionTrace`

## Workflow (строгий)

### Step A — Routing (обязательный)

1) Если во входе есть `TaskTypeOverride` (см. `references/override-contract.md`) — **не запускай** router prompt, прими override.
2) Иначе — выполни Router prompt из `references/router-prompt.md`.
3) Всегда начни ответ с `## Routing` в строгом формате (см. `references/output-pack-format.md`).

### Step B — Stop conditions (обязательный “предохранитель”)

Если `MustHaveInputs` содержит блокеры или выполняется любой `StopConditions`, то:
- **не** генерируй Spec/AC/TestPack “как будто всё известно”;
- верни только `Routing` + P0 вопросы/Unknowns + “что предоставить” + `ExecutionTrace`.

### Step C — Full pack (по умолчанию)

Если stop conditions не сработали — собери пакет артефактов по `RecommendedPrompts` (или по матрице в `references/tasktype-matrix.md` при override):
- Gate 0 → 1 → 2 → 3/3.1 (+3.2 при признаках конфликтов) → 4 → 5 → 6 → 7 → 8/8.x
- Затем валидаторы 9.1–9.3 и “очистка” результата (всё без основания → в Unknowns/Questions/Assumptions).

### Step D — Validators (обязательный)

Прогони валидаторы из `references/validators.md`:
- 9.1: галлюцинации/домыслы
- 9.2: проверяемость
- 9.3: двусмысленность

Если валидатор нашёл проблему:
- перепиши формулировку так, чтобы она либо имела основание во входе, либо стала вопросом/assumption.

### Step E — ExecutionTrace (обязательный)

Всегда заканчивай ответ секцией `## ExecutionTrace`:
- `Applied`: что реально сделано (какие гейты)
- `Skipped`: что пропущено и почему

Строгий шаблон: см. `references/output-pack-format.md`.

## Guardrails (“не выдумывать”)

Применяй к каждому артефакту:
- **только факты из входа**;
- нет фактов → **вопрос** (P0/P1/P2);
- если нужно двигаться без ответа — оформляй `ASSUMPTION` (с риском и как валидировать);
- избегай “как обычно делают”, “мы обычно храним”, “эндпоинт будет таким-то” — это запрещено без входных данных.

## Как понять, что происходит (не “магия”)

- `task-formalizer` не чёрный ящик: он всегда показывает `## Routing` (TaskType/MustHaveInputs/RecommendedPrompts/StopConditions) и `## ExecutionTrace` (что было применено/пропущено).
- TaskType router отвечает: **какие артефакты нужны** и когда остановиться (см. `references/router-prompt.md` и `references/output-pack-format.md`).

## Примеры (коротко)

### Пример 1 — Router вернул Other → override на Integration

Вход:

```text
Нужно сделать интеграцию с внешним сервисом: подключение по API key, импорт сущностей, повторная синхронизация только новых, понятные ошибки при отзыве ключа.
```

Если роутер вернул `Other`, перезапуск с override:

```text
TaskTypeOverride: Integration
```

Ожидаемое поведение:
- `## Routing` покажет `TaskType: Integration` и список `RecommendedPrompts`.
- Если не хватает определения “только новые”/ошибок/статусов — скилл вынесет это в P0/P1 вопросы и может остановиться до Spec/AC (в зависимости от MustHaveInputs/StopConditions).

### Пример 2 — SecurityAuthZ (доступы)

Вход:

```text
Нужно ограничить доступ к API: роли Admin/Member/Guest, разные права на операции над ресурсами проекта. Нужны коды ошибок, сообщения и аудит.
TaskTypeOverride: SecurityAuthZ
```

Ожидаемое поведение:
- В пакете появится матрица прав (черновик), error model (status/code/message/cta) и список P0 вопросов (scope ролей, область действия, что считать breaking change).
