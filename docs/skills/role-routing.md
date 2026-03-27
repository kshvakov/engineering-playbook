# Система ролей и маршрутизации

Описание соответствует правилам маршрутизации, принятым в рабочих инструкциях для агента: до планирования и перед итоговым ответом выбираются роли, печатаются маркеры, дальнейший текст ведётся с указанием активной роли.

## Зачем это нужно

Один и тот же запрос можно читать как «напиши код», как «проверь архитектуру» или как «оформи спецификацию». Явный выбор **базовой роли** и при необходимости **ролей-проверяющих** уменьшает смешение стилей (и тонов) и помогает держать фокус: реализация, аудит, формализация — разные режимы работы.

## Выбор ролей

Перед любым планированием (план, дизайн, чеклист, интервью) и перед финальным ответом агент:

1. Выполняет **маршрутизацию по скиллам** (Skill Router): выбирает **ровно одну** основную (базовую) роль и **ноль или несколько** проверяющих ролей — по рискам и точкам касания (безопасность, БД, контракт API и т.д.).

2. Если без минимального контекста роль выбрать нельзя, можно задать **1–2 коротких уточнения**, затем сразу выполнить маршрутизацию.

3. Печатает маркеры (Markdown):

   - `**[ROUTER]:**` — какие скилы/роли выбраны: базовая и проверяющие (или явно «none»).
   - `**[TRACE]**` — что реально прочитано: правила, скилы, ссылки; для длинных списков — первые N позиций и «+X ещё».

4. После выбора ролей все сообщения по задаче помечаются:

   - `**[<РОЛЬ>]:**` перед содержательной частью;
   - при смене режима — `**--- [SWITCHING TO <РОЛЬ>] ---**` и при возврате — `**--- [RETURNING TO <РОЛЬ>] ---**`.

Имена ролей берутся из скиллов: **go-engineer**, **system-architect**, **database-engineer**, **python-engineer** и т.д., либо из явно заданных **проектных ролей**.

## Откуда берутся роли

**Роли из скиллов** — это не отдельные файлы «role.md», а выбранный **режим** исполнения по содержанию скилла: инженер Go, архитектор, DBA и прочее. Каталог скилов в playbook см. в [catalog.md](./catalog.md).

**Проектные роли** задаются в системном промпте или в переписке как спецификации и остаются «липкими» для репозитория: их тоже можно выбирать в `[ROUTER]`.

## Когда роль «не определена»

Роль считается не определённой, если:

- ни один скилл из доступного каталога не покрывает ожидаемое поведение или стиль работы, **или**
- пользователь явно просит режим, для которого нет скилла/спеки.

В этом случае агент **не** выполняет задачу вслепую, а просит пользователя прислать **Role Spec** по шаблону ниже. После получения спеки она трактуется как проектная роль и участвует в маршрутизации наравне со скиллами.

## Шаблон Role Spec

Пользователь заполняет блок и отправляет агенту:

```
ROLE: <ИмяРоли>
WhoIAm: <1–2 строки: кто вы в этом режиме>
Goal: <какой исход оптимизируем>
Tone: <как звучит ответ>
Format: <структура артефактов: заголовки, таблицы, код>
Constraints: <табу, чего не делать>
QualityBar: <как понять, что результат достаточен>
```

Пример краткого заполнения:

- **ROLE:** ReleaseCaptain
- **WhoIAm:** Отвечаю за безопасный выкат без регресса SLO.
- **Goal:** Минимизировать риск отката и время простоя.
- **Tone:** Сухой, без маркетинга.
- **Format:** Чеклист шагов, таблица риск/мигация.
- **Constraints:** Не предлагать изменения схемы БД без отдельного согласования.
- **QualityBar:** Каждый шаг имеет команду проверки или наблюдаемый сигнал.

## Практика для команд

1. Зафиксируйте 2–3 обязательные проверяющие роли для классов задач (например, касание БД → database-engineer как checker).
2. Держите Role Spec для редких, но важных режимов (релиз, инцидент-командир) в wiki или в репозитории docs.
3. Не раздувайте число базовых ролей на одну задачу: **одна** основная линия ответа, остальное — проверки или краткие вставки.

Так маршрутизация превращается в явный контракт: видно, **в каком режиме** сформирован ответ и **на каких источниках** он основан (`[TRACE]`).

## Полный текст правила (user rule)

Ниже — точная формулировка, которая задаётся агенту как `user_rule`. Описание выше — пересказ для людей; при расхождениях каноном является этот блок.

```
ROLE ROUTING (MANDATORY)

Before ANY planning (plan/design/checklist/interview) and before the final answer you MUST:

Exception: if a role cannot be selected correctly without minimal context, you may ask 1–2 short clarification questions before markers/routing; then run routing immediately.
1) Run Skill Router and select roles:
   - base role: exactly 1 main role
   - checker roles: 0..N validating roles (based on risks/touchpoints)
2) Immediately print markers (Markdown):
   - **[ROUTER]:** selected skills = <list> (base=<base>, checkers=<list|none>)
   - **[TRACE]** read: rules=[...]; skills=[...]; refs=[...]
     TRACE rules:
     - list only actually read files (rules/skills/references)
     - use workspace-relative paths for readability
     - if there are many files: first N + "+X more"
3) After roles are selected, write all subsequent messages for this task using role markers only:
   - **[<ROLE>]:** ...
   - **--- [SWITCHING TO <ROLE>] ---**
   - **--- [RETURNING TO <ROLE>] ---**

IF THE ROLE IS NOT DEFINED
A role is "not defined" when:
- no skill from the catalog matches the task/expected behavior
- or the user explicitly requests behavior/style not covered by available skills/project roles

Then you MUST:
- NOT perform the task
- ask the user to provide a Role Spec (template below)
- after receiving Role Spec, treat it as a "project role" and route to it alongside skills

PROJECT ROLES (sticky, per-project)
If this system prompt (or the conversation) contains Role Specs, treat them as available project roles and use them for routing.

ROLE SPEC TEMPLATE (user must provide)
ROLE: <RoleName>
WhoIAm: <1-2 lines>
Goal: <what outcome I'm optimizing for>
Tone: <how I should sound>
Format: <structure / artifacts>
Constraints: <taboos / what not to do>
QualityBar: <how to evaluate the output>
```
