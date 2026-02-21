# Workplace Init Bundle

Инструментарий для автоматической генерации стандартизированных рабочих пространств (workplace) — Git-репозиториев с единой структурой документации, скриптами автоматизации и стандартами разработки.

---

## Концепция

**Workplace** — это репозиторий проекта, организованный по принципу разделения кода и документации:

```text
workplace/
├── knowledge/      # Документация: спецификации, стандарты, история решений
│   ├── services/   # Спецификации сервисов
│   ├── features/   # Спецификации фичей (Hub-and-Node паттерн)
│   ├── standards/  # Стандарты разработки и документирования
│   └── memory/     # История решений агентов (опционально)
├── projects/       # Исходный код, конфигурации деплоя
├── scripts/        # Автоматизация (bootstrap, backup, cleanup)
└── .data/          # Локальные данные, бандлы (игнорируется Git)
```

Ключевое правило: **документация живёт в `knowledge/`, код — в `projects/`**. Смешение запрещено.

### Hub-and-Node

Фичи, затрагивающие несколько сервисов, документируются в два уровня:

- **Hub** (`knowledge/features/<feature>/`) — зачем и что: бизнес-задача, архитектурные решения, data flow, blast radius
- **Node** (`knowledge/services/<service>/features/<feature>/`) — как конкретный сервис реализует свою часть

### Поддерживаемые типы проектов

| Тип        | Описание                                             |
| :--------- | :--------------------------------------------------- |
| `code`     | Сервисы на одном языке (Java, Kotlin, TypeScript...) |
| `infra`    | Infrastructure as Code (Nomad, Docker Compose, K8s)  |
| `monorepo` | Полиглотный проект с несколькими подпроектами        |

---

## Быстрый старт

### 1. Подготовка

Скопируй `.data/workplace-init-bundle/` в корень нового пустого репозитория:

```bash
mkdir my-project && cd my-project
git init

# Скопировать бандл (адаптируй путь)
cp -r /path/to/workplace-init-bundle .data/workplace-init-bundle
```

### 2. Генерация

Открой AI-агент (Claude, Gemini или аналог) в директории проекта и передай промпт:

```text
Прочитай .data/workplace-init-bundle/WORKPLACE_GENERATION_PROMPT.md и выполни генерацию workplace.
```

Агент проведёт через 5 фаз с контрольными точками:

1. **Configuration** — сбор параметров проекта (код, тип, язык, оркестратор)
2. **Scaffold** — создание структуры директорий, копирование стандартов и скриптов
3. **Documentation** — генерация README.md и AGENTS.md
4. **Projects** — инициализация подпроектов из `projects_list`
5. **Finalization** — валидация структуры и начальный коммит

### 3. После генерации

```bash
cp .env.example .env              # Настроить переменные окружения
./scripts/workplace-bootstrap.sh  # Проверить и доинициализировать
```

---

## Структура бандла

```text
.data/workplace-init-bundle/
├── WORKPLACE_GENERATION_PROMPT.md   # Промпт для AI-агента
├── templates/                       # Шаблоны файлов (параметризуются через {{PLACEHOLDER}})
│   ├── AGENTS-{code,infra,monorepo}.md.template
│   ├── README.md.template
│   ├── Makefile-{code,infra,monorepo}.template
│   ├── gitignore-{code,infra,monorepo}.template
│   ├── gitattributes.template
│   ├── markdownlint.yml
│   ├── env-example.template
│   ├── knowledge-index.md.template
│   ├── services-index.md.template
│   ├── features-index.md.template
│   └── service-stub.md.template
├── standards/                       # Стандарты документации (копируются as-is)
│   ├── INDEX.md
│   ├── standard-specification-common-format.md
│   ├── standard-service-specification.md
│   ├── standard-feature-specification.md
│   ├── standard-service-dependencies.md
│   └── standard-agent-memory-format.md
└── scripts/                         # Шаблоны скриптов автоматизации
    ├── workplace-bootstrap.sh.template
    ├── workplace-cleanup.sh.template
    ├── workplace-backup.sh.template
    ├── workplace-restore.sh.template
    └── workplace-print-dir.sh.template
```

### Шаблонизация

Шаблоны используют placeholder-ы в формате `{{PLACEHOLDER}}`, которые заменяются при генерации:

| Placeholder        | Описание                  | Пример           |
| :----------------- | :------------------------ | :--------------- |
| `{{PROJECT_CODE}}` | Короткий код проекта      | `my-project`     |
| `{{PROJECT_NAME}}` | Полное название           | `My Project`     |
| `{{PROJECT_TYPE}}` | Тип проекта               | `code`           |
| `{{LANGUAGE}}`     | Основной язык             | `java`           |
| `{{BUILD_TOOL}}`   | Система сборки            | `gradle`         |
| `{{ORCHESTRATOR}}` | Оркестратор               | `docker-compose` |
| `{{SERVICE_NAME}}` | Имя сервиса (для stub-ов) | `My Service`     |
| `{{SERVICE_CODE}}` | Код сервиса (для stub-ов) | `my-service`     |

---

## Стандарты

Бандл включает 5 стандартов документации:

| Стандарт                               | Назначение                                                      |
| :------------------------------------- | :-------------------------------------------------------------- |
| `standard-specification-common-format` | Общий формат документов: H1 с emoji, backlink, doc-deps, ссылки |
| `standard-service-specification`       | Структура спецификации сервиса: цель, API, зависимости, деплой  |
| `standard-feature-specification`       | Hub-and-Node паттерн для фичей                                  |
| `standard-service-dependencies`        | Типы зависимостей, blast radius, порядок деплоя, API контракты  |
| `standard-agent-memory-format`         | Формат memory-файлов для фиксации решений агентов (опционально) |

Стандарты универсальны и не привязаны к конкретному языку, фреймворку или оркестратору.

---

## Расширение

### Новый тип проекта

1. Создать `templates/gitignore-<type>.template`
2. Создать `templates/Makefile-<type>.template`
3. Создать `templates/AGENTS-<type>.md.template`
4. Добавить тип в таблицу стеков в промпте

### Новый язык/стек

1. Добавить запись в таблицу "Поддерживаемые стеки" в промпте
2. Описать структуру проекта в Фазе 4.1
3. Дополнить AGENTS-шаблон секцией Coding Style

### Новый стандарт

1. Создать файл в `standards/`
2. Обновить `standards/INDEX.md`
3. Добавить шаг копирования в Фазу 2.3 промпта

---

## Скрипты

Генерируемый workplace включает набор идемпотентных скриптов:

| Скрипт                   | Назначение                                            |
| :----------------------- | :---------------------------------------------------- |
| `workplace-bootstrap.sh` | Инициализация: проверка зависимостей, создание папок  |
| `workplace-cleanup.sh`   | Очистка артефактов сборки и логов (`--dry-run`)       |
| `workplace-backup.sh`    | Архивация workplace (tar.gz/zip) с контрольной суммой |
| `workplace-restore.sh`   | Восстановление из архива с проверкой целостности      |
| `workplace-print-dir.sh` | Определение корня workplace                           |
