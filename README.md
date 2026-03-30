# Engineering Playbook

Практическое руководство по использованию AI-ассистентов в повседневной разработке и эксплуатации. Основано на реальном опыте работы над production-сервисами Kinescope.

Это не теория и не маркетинг. Здесь описан конкретный рабочий процесс, в котором AI — не «автодополнение кода», а полноценный участник цикла: от формализации задачи до code review и деплоя, от алерта мониторинга до диагностики сервера и capacity-аудита.

## Для кого

Для разработчиков и инженеров эксплуатации, которые хотят перейти от «спросил ChatGPT — скопировал ответ» к системному использованию AI в рабочем процессе. Не нужен опыт работы с AI-инструментами — playbook проведёт от установки до первого merge request или разбора инцидента.

## Быстрый старт

1. [Установка и настройка](docs/setup.md) — Cursor, MCP-серверы, скилы
2. [Первый рабочий день с AI](docs/first-day.md) — пошаговый сценарий
3. [Обязательные скилы](docs/must-have-skills.md) — минимальный набор для старта

## Оглавление

### Рабочий процесс

- [Полный цикл работы](docs/workflow/overview.md) — от задачи до merge
- [Формализация задач](docs/workflow/task-formalization.md) — превращение «хочу фичу» в spec
- [Планирование](docs/workflow/planning.md) — написание и выполнение планов
- [Мультирепозиторная работа](docs/workflow/multi-repo.md) — backend + frontend одновременно
- [Дебаг с AI](docs/workflow/debugging.md) — системный подход к поиску ошибок
- [Code Review](docs/workflow/code-review.md) — AI как ревьюер
- [Ops и инфраструктура](docs/workflow/ops-infrastructure.md) — от алерта до рекомендации

### MCP-интеграции

- [Что такое MCP](docs/mcp/overview.md) — зачем AI нужен доступ к инфраструктуре
- [Таск-трекер](docs/mcp/task-tracker.md) — создание задач, привязка веток
- [Мессенджер](docs/mcp/messenger.md) — контекст обсуждений из Пачки
- [Базы данных](docs/mcp/databases.md) — PostgreSQL и ClickHouse из IDE
- [Инфраструктура](docs/mcp/infrastructure.md) — Proxmox, серверы, capacity из IDE

### Паттерны

- [Параллельные агенты](docs/patterns/parallel-agents.md) — несколько задач одновременно
- [План → Выполнение](docs/patterns/plan-execute.md) — основной паттерн работы
- [Brainstorming → Дизайн](docs/patterns/brainstorming-design.md) — проектирование через развилки
- [Привязка веток к задачам](docs/patterns/kaiten-branches.md) — автоматическая трекабильность
- [Верификация](docs/patterns/verification.md) — доказательства вместо утверждений
- [Триаж алертов](docs/patterns/alert-triage.md) — алерт → диагностика → действие
- [SSH-диагностика из IDE](docs/patterns/ssh-diagnostics.md) — серверы из Cursor

### Кейсы из практики

- [Диск заполнен: deleted files + RAID](docs/cases/disk-space-alert.md) — диагностика диска и аудит оборудования
- [Аудит Proxmox: VM и capacity](docs/cases/proxmox-capacity-audit.md) — инвентаризация и capacity planning
- [429 на странице плейлиста: виртуализация списка](docs/cases/playlist-poster-429.md) — от сообщения в мессенджере до фикса за одну сессию
- [Рефакторинг ошибок плеера](docs/cases/player-error-handling-refactor.md) — от жалобы на битый чанк до аудита 3 драйверов и 8 задач в Kaiten
- [Ревью кастомных колонок каталога](docs/cases/catalog-custom-columns-review.md) — AI-ревью ветки gateway: SQL-производительность, scope ветки, мёртвые поля в контракте
- [GPU «fallen off the bus» после зависания](docs/cases/encoder-gpu-fallen-off-bus.md) — диф-диагностика энкодера по USE: GPU Xid 79, каскад проблем, изоляция причины

### Скилы

- [Каталог скилов](docs/skills/catalog.md) — что есть и для чего
- [Как писать скилы](docs/skills/how-to-write.md) — создание своих инструкций для AI
- [Система ролей](docs/skills/role-routing.md) — маршрутизация по ролям

### Прочее

- [Антипаттерны](docs/antipatterns.md) — что не работает и почему

## Скилы

Директория [skills/](skills/) содержит полные копии must-have скилов, готовых к использованию. Скопируйте их в `~/.cursor/skills/` и они сразу заработают в Cursor.
