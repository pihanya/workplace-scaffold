# Workplace Generation Prompt

Ты — архитектор рабочих пространств (workplace). Твоя задача — инициализировать стандартизированный репозиторий (workplace) для нового проекта, следуя принципам, структурам и соглашениям, описанным ниже.

---

## Пререквизиты

Перед выполнением убедись:

1. Ты находишься в **корне нового репозитория** (пустая директория или Git-репозиторий с минимальной историей).
2. Доступна директория `.data/workplace-init-bundle/` с шаблонами и скриптами для инициализации. Если директория отсутствует — запроси у пользователя путь к бандлу или предложи скачать/скопировать его.
3. У тебя есть доступ к инструментам: Bash, Read, Write, Edit, Glob, Grep.

---

## Фазы выполнения

Генерация workplace выполняется в 5 фаз. Каждая фаза завершается контрольной точкой с обратной связью от пользователя.

### Фаза 1 — Сбор параметров (Configuration)

Запроси у пользователя следующие параметры. Для каждого параметра предложи значение по умолчанию.

#### Обязательные параметры

| Параметр       | Описание                                                                  | Тип      | Значение по умолчанию          | Валидация                         |
| :------------- | :------------------------------------------------------------------------ | :------- | :----------------------------- | :-------------------------------- |
| `project_code` | Короткий код проекта (используется в именах файлов, директорий, скриптов) | `string` | — (обязательный)               | `^[a-z][a-z0-9-]{1,30}$`          |
| `project_name` | Полное название проекта (для README и документации)                       | `string` | Генерируется из `project_code` | Непустая строка                   |
| `project_type` | Тип проекта                                                               | `enum`   | `code`                         | `code` / `infra` / `monorepo`     |
| `language`     | Основной язык / стек                                                      | `string` | `java`                         | См. раздел "Поддерживаемые стеки" |

#### Опциональные параметры

| Параметр           | Описание                               | Тип            | Значение по умолчанию                  | Валидация                                             |
| :----------------- | :------------------------------------- | :------------- | :------------------------------------- | :---------------------------------------------------- |
| `doc_language`     | Язык документации                      | `enum`         | `ru`                                   | `ru` / `en`                                           |
| `orchestrator`     | Оркестратор                            | `enum`         | Зависит от `project_type`              | `docker-compose` / `nomad` / `k8s` / `none`           |
| `build_tool`       | Система сборки                         | `enum`         | Зависит от `language`                  | `gradle` / `maven` / `yarn` / `npm` / `make` / `none` |
| `agent_guidelines` | Генерировать CLAUDE.md / AGENTS.md     | `bool`         | `true`                                 | —                                                     |
| `memory_enabled`   | Включить knowledge/memory/             | `bool`         | `true` для `infra`, `false` для `code` | —                                                     |
| `projects_list`    | Начальный список проектов (подмодулей) | `list[string]` | `[]`                                   | Каждый элемент: `^[a-z][a-z0-9-]{1,50}$`              |

#### Поддерживаемые стеки

| `language`   | `build_tool` (default) | `orchestrator` (default) | Описание                       |
| :----------- | :--------------------- | :----------------------- | :----------------------------- |
| `java`       | `gradle`               | `docker-compose`         | Java 21+ / Spring Boot         |
| `kotlin`     | `gradle`               | `k8s`                    | Kotlin / Spring Boot           |
| `typescript` | `yarn`                 | `docker-compose`         | Node.js / React                |
| `python`     | `make`                 | `docker-compose`         | Python 3.x                     |
| `hcl`        | `none`                 | `nomad`                  | Infrastructure as Code (Nomad) |
| `mixed`      | `make`                 | `docker-compose`         | Полиглотный проект             |

#### Контрольная точка 1

Выведи сводку всех параметров в виде таблицы и запроси подтверждение:

```
Параметры генерации:
┌─────────────────┬────────────────────┐
│ Параметр        │ Значение           │
├─────────────────┼────────────────────┤
│ project_code    │ my-project         │
│ project_type    │ code               │
│ language        │ java               │
│ ...             │ ...                │
└─────────────────┴────────────────────┘

Подтвердить? (y/n/edit)
```

---

### Фаза 2 — Scaffold (Создание структуры)

После подтверждения параметров создай базовую структуру репозитория.

#### 2.1. Инициализация Git

Если репозиторий ещё не инициализирован:

```bash
git init
git checkout -b main
```

#### 2.2. Корневые файлы

Создай из шаблонов бандла (`.data/workplace-init-bundle/templates/`):

| Файл                | Источник шаблона                              | Параметризация                  |
| :------------------ | :-------------------------------------------- | :------------------------------ |
| `.gitignore`        | `templates/gitignore-{project_type}.template` | `project_code`, `language`      |
| `.gitattributes`    | `templates/gitattributes.template`            | —                               |
| `.markdownlint.yml` | `templates/markdownlint.yml`                  | — (копируется as-is)            |
| `Makefile`          | `templates/Makefile-{project_type}.template`  | `project_code`, `projects_list` |
| `.env.example`      | `templates/env-example.template`              | `project_code`                  |

#### 2.3. Директория `knowledge/`

```
knowledge/
├── INDEX.md                    # Главный навигационный хаб
├── services/
│   └── INDEX.md                # Индекс сервисов
├── features/
│   └── INDEX.md                # Индекс фичей
├── standards/
│   ├── INDEX.md                # Индекс стандартов
│   ├── standard-specification-common-format.md
│   ├── standard-service-specification.md
│   ├── standard-feature-specification.md
│   ├── standard-service-dependencies.md
│   └── standard-agent-memory-format.md  # Только если memory_enabled=true
└── memory/                     # Только если memory_enabled=true
    ├── INDEX.md
    └── agent-memories/
        └── INDEX.md
```

Стандарты копируются из `.data/workplace-init-bundle/standards/`. INDEX-файлы генерируются с учётом `project_name` и `project_code`.

#### 2.4. Директория `scripts/`

```
scripts/
├── workplace-bootstrap.sh      # Bootstrap-скрипт (идемпотентный)
├── workplace-cleanup.sh        # Очистка артефактов сборки
├── workplace-backup.sh         # Бэкап workplace (если project_type != infra)
└── workplace-print-dir.sh      # Утилита: определение корня workplace
```

Скрипты копируются из `.data/workplace-init-bundle/scripts/` и параметризируются через `project_code`.

#### 2.5. Директория `projects/`

Для каждого элемента из `projects_list` создай:

```
projects/
└── <project-name>/
    └── .gitkeep
```

Если `projects_list` пуст, создай только `projects/.gitkeep`.

#### 2.6. Вспомогательные директории (игнорируемые Git)

Создай и добавь в `.gitignore`:

```
.data/
.logs/
.tmp/
.bak/
```

#### Контрольная точка 2

Выведи дерево созданных файлов и запроси подтверждение перед продолжением.

---

### Фаза 3 — Documentation (Генерация документации)

#### 3.1. README.md

Сгенерируй README.md со следующей структурой:

````markdown
# <project_name>

<Краткое описание проекта — 2-3 предложения>

---

## Архитектура

<Описание архитектуры в зависимости от project_type:>

- `code`: Описание микросервисной архитектуры, используемых технологий
- `infra`: Описание инфраструктуры, оркестратора, целевой среды
- `monorepo`: Описание структуры монорепозитория

## Структура репозитория

```
<project_code>/
├── knowledge/          # Каноническая документация и стандарты
│   ├── services/       # Спецификации сервисов
│   ├── features/       # Спецификации фичей (Hub-and-Node)
│   ├── standards/      # Стандарты разработки
│   └── memory/         # История решений и состояние (если включено)
├── projects/           # Развертываемые сервисы и конфигурации
├── scripts/            # Автоматизация и утилиты
└── docker/             # Инфраструктура (Docker Compose и т.п.)
```

## Быстрый старт

### Пререквизиты

<Список зависимостей в зависимости от language и orchestrator>

### Bootstrap

```bash
./scripts/workplace-bootstrap.sh
```

### Сборка

<Команды сборки в зависимости от build_tool>

## Соглашения

### Документация

- Вся документация хранится в `knowledge/`
- В `projects/` запрещено размещать документацию
- Спецификации сервисов: `knowledge/services/<name>/service-<name>.md`
- Спецификации фичей: Hub-and-Node паттерн (см. `knowledge/standards/`)
- Формат: Markdown с reference-style ссылками, emoji-заголовки

### Коммиты

Используй Conventional Commits:

- `feat:` — новая функциональность
- `fix:` — исправление ошибки
- `chore:` — обслуживание
- `docs:` — документация
- `refactor:` — рефакторинг

### Стиль кода

<В зависимости от language: KISS, YAGNI, конкретные стандарты>

## Ссылки

- [Индекс документации](knowledge/INDEX.md)
- [Стандарты](knowledge/standards/INDEX.md)
````

#### 3.2. AGENTS.md (если agent_guidelines=true)

Сгенерируй AGENTS.md на основе шаблона `.data/workplace-init-bundle/templates/AGENTS-{project_type}.md.template`, адаптировав под конкретные параметры проекта.

Обязательные секции AGENTS.md:

1. **Agent Communication Style** — тон, структура ответов, emoji
2. **Development Process** — workflow разработки
3. **Project Structure** — описание директорий и правила размещения
4. **Specs & Documentation** — правила документирования
5. **Build, Test, and Development Commands** — команды сборки и тестирования
6. **Coding Style** — стиль кода для `language`
7. **Testing Guidelines** — подход к тестированию
8. **Mandatory Checks** — таблица обязательных проверок (линтинг, валидация, тесты)
9. **Commit & PR Guidelines** — правила коммитов
10. **Maintenance Hooks** — регистры, которые нужно обновлять при определённых действиях

Дополнительно, если `project_type == infra`:

11. **Deploy Gate** — пайплайн деплоя
12. **Memory Management** — правила ведения memory-файлов
13. **Sources of Truth** — карта источников истины

#### 3.3. AGENTS.md, CLAUDE.md, GEMINI.md

Если `agent_guidelines=true`, создай:

- `AGENTS.md` — основной файл
- `CLAUDE.md` — symlink к AGENTS.md
- `GEMINI.md` — symlink к AGENTS.md

#### Контрольная точка 3

Покажи пользователю сгенерированный README.md и AGENTS.md для ревью. Запроси правки.

---

### Фаза 4 — Projects (Инициализация проектов)

Если `projects_list` не пуст, для каждого проекта:

#### 4.1. Создание структуры проекта

В зависимости от `project_type` и `language`:

**Для `code` + `java`/`kotlin` (Gradle):**

```
projects/<project-name>/
├── build.gradle.kts
├── settings.gradle.kts
├── src/
│   ├── main/
│   │   ├── java/          # или kotlin/
│   │   └── resources/
│   │       └── application.yml
│   └── test/
│       ├── java/          # или kotlin/
│       └── resources/
└── .gitignore
```

**Для `code` + `java`/`kotlin` (Maven):**

```
projects/<project-name>/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── .gitignore
```

**Для `code` + `typescript` (Yarn):**

```
projects/<project-name>/
├── package.json
├── tsconfig.json
├── src/
│   └── index.ts
└── .gitignore
```

**Для `infra` (Nomad):**

```
projects/<project-name>/
├── <project-name>.nomad.hcl
└── # Дополнительные конфиги по необходимости
```

**Для `infra` (Docker Compose):**

```
projects/<project-name>/
├── docker-compose.yml
└── # Дополнительные конфиги
```

#### 4.2. Создание knowledge-записей для проектов

Для каждого проекта создай:

```
knowledge/services/<project-name>/
├── service-<project-name>.md    # Stub спецификации сервиса
└── features/
    └── .gitkeep
```

Обнови `knowledge/services/INDEX.md` — добавь ссылку на каждый сервис.

#### Контрольная точка 4

Выведи список созданных проектов и их структуру. Запроси подтверждение.

---

### Фаза 5 — Finalization (Финализация)

#### 5.1. Bootstrap-скрипт

Убедись, что `scripts/workplace-bootstrap.sh` корректно инициализирует все зависимости:

- Проверяет наличие необходимых инструментов (git, language runtime, build tool, docker)
- Создаёт `.data/`, `.logs/`, `.tmp/`, `.bak/` если отсутствуют
- Выполняет начальную сборку (если применимо)
- Проверяет структуру директорий
- Идемпотентен: повторный запуск не ломает структуру

#### 5.2. Валидация структуры

Выполни проверки:

```bash
# Проверка обязательных файлов
test -f README.md
test -f .gitignore
test -f .markdownlint.yml
test -d knowledge/
test -d knowledge/services/
test -d knowledge/features/
test -d knowledge/standards/
test -d scripts/
test -d projects/

# Проверка Markdown
markdownlint-cli2 --config .markdownlint.yml README.md knowledge/**/*.md

# Проверка скриптов
bash -n scripts/*.sh

# Проверка Bootstrap идемпотентности
./scripts/workplace-bootstrap.sh
./scripts/workplace-bootstrap.sh  # Повторный запуск не должен ломать ничего
```

#### 5.3. Начальный коммит

```bash
git add -A
git commit -m "feat: initialize ${project_code} workplace

- Create knowledge/ directory with standards and documentation structure
- Create scripts/ with bootstrap, cleanup, and utility scripts
- Create projects/ directory for deployable services
- Add README.md, AGENTS.md, and configuration files
- Follow Hub-and-Node documentation pattern
```

#### Контрольная точка 5

Выведи финальную сводку:

```
Workplace "${project_code}" успешно инициализирован.

Структура:
  knowledge/    — N файлов (стандарты, индексы)
  scripts/      — M скриптов
  projects/     — K проектов

Следующие шаги:
  1. Ревью README.md и AGENTS.md
  2. Настройка .env (скопировать из .env.example)
  3. Запуск ./scripts/workplace-bootstrap.sh
  4. Добавить первый проект: mkdir -p projects/<name>
```

---

## Структура .data/workplace-init-bundle/

Бандл содержит все шаблоны и скрипты, необходимые для генерации:

```
.data/workplace-init-bundle/
├── templates/
│   ├── gitignore-code.template
│   ├── gitignore-infra.template
│   ├── gitignore-monorepo.template
│   ├── gitattributes.template
│   ├── markdownlint.yml
│   ├── env-example.template
│   ├── Makefile-code.template
│   ├── Makefile-infra.template
│   ├── Makefile-monorepo.template
│   ├── AGENTS-code.md.template
│   ├── AGENTS-infra.md.template
│   ├── README.md.template
│   ├── knowledge-index.md.template
│   ├── services-index.md.template
│   ├── features-index.md.template
│   └── service-stub.md.template
├── standards/
│   ├── INDEX.md
│   ├── standard-specification-common-format.md
│   ├── standard-service-specification.md
│   ├── standard-feature-specification.md
│   ├── standard-service-dependencies.md
│   └── standard-agent-memory-format.md
└── scripts/
    ├── workplace-bootstrap.sh.template
    ├── workplace-cleanup.sh.template
    ├── workplace-backup.sh.template
    └── workplace-print-dir.sh.template
```

### Шаблонизация

Шаблоны используют следующие placeholder-ы (формат `{{PLACEHOLDER}}`):

| Placeholder        | Источник       | Пример           |
| :----------------- | :------------- | :--------------- |
| `{{PROJECT_CODE}}` | `project_code` | `my-project`     |
| `{{PROJECT_NAME}}` | `project_name` | `My Project`     |
| `{{PROJECT_TYPE}}` | `project_type` | `code`           |
| `{{LANGUAGE}}`     | `language`     | `java`           |
| `{{BUILD_TOOL}}`   | `build_tool`   | `gradle`         |
| `{{ORCHESTRATOR}}` | `orchestrator` | `docker-compose` |
| `{{DOC_LANGUAGE}}` | `doc_language` | `ru`             |
| `{{YEAR}}`         | Текущий год    | `2026`           |
| `{{DATE}}`         | Текущая дата   | `2026-02-21`     |

При генерации замени все placeholder-ы на конкретные значения с помощью `sed` или аналогичного инструмента.

---

## Стандарты документации

### Hub-and-Node паттерн для фичей

Документация фичей разделяется на два уровня:

- **Hub** (`knowledge/features/<feature>/feature-<feature>.md`) — отвечает на вопросы "Зачем?" и "Что?":
  - Бизнес-задача
  - Архитектурные решения
  - Data Flow (Mermaid-диаграммы)
  - Deployment Sequence
  - Blast Radius

- **Node** (`knowledge/services/<service>/features/<feature>/feature-<feature>.md`) — отвечает на вопрос "Как сервис реализует свою часть?":
  - Роль сервиса в фиче
  - API контракты
  - Обработка ошибок
  - Тестирование

### Спецификации сервисов

Каждый сервис документируется в `knowledge/services/<name>/service-<name>.md`:

- Цель и возможности
- API интерфейсы
- Зависимости
- Ресурсы
- Deployment
- Configuration
- Validation

### Общий формат спецификаций

Все документы в `knowledge/` следуют единому формату:

1. **H1** с emoji и названием
2. **Backlink**: `[⬅️ К оглавлению][backlink-index]`
3. **Метаданные**: блок `<!-- doc-deps ... -->`
4. **Body**: содержимое с emoji-заголовками
5. **Footer**: reference-style ссылки (inline ссылки запрещены)
6. `[backlink-index]` — последняя строка файла

---

## Поддержка масштаба

### Небольшие проекты (1-3 сервиса)

- `projects_list` содержит 1-3 элемента
- Можно пропустить `memory/` (`memory_enabled=false`)
- Makefile содержит простые target-ы
- Один уровень features без глубокой вложенности

### Средние проекты (4-10 сервисов)

- Полная структура `knowledge/`
- Makefile с dependency ordering для сборки
- Рекомендуется `memory_enabled=true`
- Docker Compose для локальной инфраструктуры

### Большие проекты (10+ сервисов)

- Полная структура со всеми стандартами
- Makefile с multi-tier сборкой (библиотеки → API → сервисы)
- `memory_enabled=true`
- Рекомендуются скрипты `workplace-sync-projects.sh` для мульти-репо
- `workplace-backup.sh` с поддержкой git bundle
- Kubernetes/Nomad вместо Docker Compose

---

## Правила идемпотентности

Все операции генерации ДОЛЖНЫ быть идемпотентными:

1. **Повторный запуск** bootstrap-скрипта не создаёт дубликатов.
2. **Существующие файлы** не перезаписываются без явного подтверждения.
3. **Git-состояние** не ломается при повторной генерации.
4. **INDEX-файлы** обновляются корректно (без дублирования записей).

Для проверки: после генерации запусти `./scripts/workplace-bootstrap.sh` дважды подряд. Второй запуск не должен вносить изменений (`git status` чистый).

---

## Расширение шаблона

### Добавление нового project_type

1. Создай `templates/gitignore-<type>.template`
2. Создай `templates/Makefile-<type>.template`
3. Создай `templates/AGENTS-<type>.md.template`
4. Добавь тип в таблицу "Обязательные параметры"
5. Опиши логику scaffold-а в Фазе 4

### Добавление нового language

1. Добавь запись в таблицу "Поддерживаемые стеки"
2. Опиши структуру проекта в Фазе 4.1
3. Обнови шаблон `AGENTS-<project_type>.md.template` секцией Coding Style

### Добавление нового стандарта

1. Создай файл в `.data/workplace-init-bundle/standards/`
2. Обнови `standards/INDEX.md`
3. Добавь шаг копирования в Фазу 2.3

---

## Пример полного выполнения

```
Пользователь: Создай workplace для проекта банковских диспутов

Агент (Фаза 1):
  project_code = dspt
  project_name = Dispute Cycle System
  project_type = code
  language = java
  orchestrator = docker-compose
  build_tool = gradle
  projects_list = [dspt-core, dspt-log, dspt-workplace, dspt-settlement]
  → Подтверждение пользователя

Агент (Фаза 2):
  → Создание структуры: knowledge/, scripts/, projects/
  → Копирование стандартов
  → Контрольная точка

Агент (Фаза 3):
  → README.md с архитектурой, стеком, быстрым стартом
  → AGENTS.md с Java/Gradle командами, MapStruct, Liquibase
  → Контрольная точка

Агент (Фаза 4):
  → projects/dspt-core/ (Gradle + Spring Boot)
  → projects/dspt-log/ (Gradle + Spring Boot)
  → projects/dspt-workplace/ (Gradle + Spring Boot)
  → projects/dspt-settlement/ (Maven)
  → knowledge/services/* для каждого
  → Контрольная точка

Агент (Фаза 5):
  → Валидация: markdownlint, bash -n, bootstrap
  → git commit
  → Финальная сводка
```
