# 📋 Стандарт Формата Memory-файлов Агента

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: standard-agent-memory-format
depends_on:
  - knowledge/standards/standard-specification-common-format.md
  - knowledge/standards/standard-temp-files-organization.md
provides_for:
  - knowledge/memory/agent-memories/INDEX.md
-->

## 📘 Цель и Область Применения

Этот стандарт определяет единый формат для memory-файлов агента, хранящихся в `knowledge/memory/agent-memories/`.

**Цели стандарта:**

| Цель                            | Описание                                                                  |
| ------------------------------- | ------------------------------------------------------------------------- |
| **Единообразие**                | Консистентный формат для всех записей значимых решений и действий         |
| **Поиск и анализ**              | Структурированные метаданные позволяют быстро находить релевантные записи |
| **Аудит решений**               | История изменений с контекстом для понимания эволюции системы             |
| **Передача контекста**          | Передача знаний между сессиями агента и команды разработчиков             |
| **Документирование инцидентов** | Запись проблем и их решений для будущих агентов                           |

**Область применения:**

- Все файлы в директории `knowledge/memory/agent-memories/`
- Значимые решения, изменения конфигурации, деплои, инциденты
- Аудитория: AI-агенты и разработчики/операторы

Memory-файлы **НЕ** заменяют service specs или feature specs — они дополняют их, фиксируя историю принятия решений и операционные детали, которые не входят в основную документацию.

---

## 📂 Структура Файла

Каждый memory-файл должен следовать этой структуре:

````markdown
# <Title>

- **Date:** YYYY-MM-DD HH:MM (UTC)
- **Type:** <deployment|installation|configuration|tuning|rollout|documentation|incident|decision|rollback>
- **Action:** <Что было сделано>
- **Status:** <completed|in_progress|rolled_back>

## 🎯 Context

<Контекст и причины>

## 🔧 Changes

<Конкретные изменения>

## ✅ Validation

<Как проверено>

## 💥 Impact

<Что затронуто>

## 📚 References

<Ссылки на связанные документы>
````

**Правила:**

1. **Заголовок H1** — краткое описание действия (без emoji)
2. **Метаданные в начале** — дата, тип, action, статус
3. **Разделы с emoji** — Context, Changes, Validation, Impact, References
4. **Reference-ссылки** — в footer, inline-ссылки запрещены

---

## 🧠 Типы Memory-файлов

| Тип               | Описание                                  | Когда создавать                                      | Пример                            |
| ----------------- | ----------------------------------------- | ---------------------------------------------------- | --------------------------------- |
| **Deployment**    | Деплой сервиса или изменение конфигурации | После деплоя сервиса или изменения конфигурации      | `service-deployment`              |
| **Installation**  | Установка нового ПО/пакета                | После установки нового инструмента на хост           | `monitoring-tool-installation`    |
| **Configuration** | Изменение существующей конфигурации       | После правки конфигов без деплоя                     | `orchestrator-config-update`      |
| **Tuning**        | Оптимизация производительности            | После изменения ресурсов или параметров              | `search-service-cpu-tuning`       |
| **Rollout**       | Запуск фичи в production                  | После включения фичи для пользователей               | `feature-x-rollout`               |
| **Rollback**      | Откат изменений                           | После отката проблемного деплоя                      | `service-y-rollback`              |
| **Documentation** | Документирование процессов                | После фиксации команд, процедур                      | `benchmark-commands-doc`          |
| **Incident**      | Инцидент и его разрешение                 | После resolution инцидента                           | `outage-2024-01-15`               |
| **Decision**      | Архитектурное/значимое решение            | После принятия важного решения                       | `storage-migration-decision`      |

**Правила выбора типа:**

- Если сомневаетесь между двумя типами — выбирайте более специфичный
- Rollback — отдельный тип, а не статус Deployment
- Decision — для принципиальных решений, влияющих на архитектуру

---

## 📋 Обязательные Поля

### Метаданные (в начале файла)

| Поле       | Формат                                      | Пример                                   | Описание                         |
| ---------- | ------------------------------------------- | ---------------------------------------- | -------------------------------- |
| **Date**   | `YYYY-MM-DD HH:MM (UTC)`                    | `2026-02-08 11:05 (UTC)`                 | Время выполнения действия        |
| **Type**   | Один из типов выше                          | `deployment`                             | Категория события                |
| **Action** | Глагол в прошедшем времени                  | `Deployed updated service configuration` | Что было сделано (1 предложение) |
| **Status** | `completed` / `in_progress` / `rolled_back` | `completed`                              | Текущий статус                   |

### Обязательные разделы

| Раздел         | Emoji | Содержание                                     | Минимум              |
| -------------- | ----- | ---------------------------------------------- | -------------------- |
| **Context**    | 🎯    | Почему это было сделано, какой контекст        | 2-3 предложения      |
| **Changes**    | 🔧    | Конкретные изменения (файлы, конфиги, команды) | Список или команды   |
| **Validation** | ✅    | Как проверено, что всё работает                | Команды или проверки |
| **Impact**     | 💥    | Что затронуто, side effects                    | Компоненты/сервисы   |

### Язык записей

- `MUST`: H1, поле `Action` и содержимое разделов (`Context`, `Changes`,
  `Validation`, `Impact`, `References`) оформляются на языке документации
  проекта (`doc_language`: `ru` или `en`).
- `SHOULD`: использовать единый стиль формулировок в пределах одного
  репозитория.
- Технические идентификаторы сохраняются без перевода: команды, пути, имена
  сервисов/процессов, цитаты из логов, enum-значения (`Type`, `Status`) и
  другие неизменяемые системные токены.

### Опциональные разделы

| Раздел            | Когда включать                 | Содержание                             |
| ----------------- | ------------------------------ | -------------------------------------- |
| **References**    | Всегда при наличии связей      | Ссылки на service specs, feature specs |
| **Commands**      | Для Installation/Documentation | Ключевые команды с описанием           |
| **Results**       | Для Tuning/Decision            | Измерения, метрики, сравнения          |
| **Rollback Plan** | Для Deployment/Rollout         | План отката на случай проблем          |

---

## 🔗 Интеграция с Документацией

Memory-файлы **должны** содержать ссылки на связанные документы для трассируемости.

### Когда ссылаться

| Тип Memory    | Ссылка на                       | Пример                                                 |
| ------------- | ------------------------------- | ------------------------------------------------------ |
| Deployment    | Service Spec                    | `knowledge/services/my-service/service-my-service.md`  |
| Rollout       | Feature Hub                     | `knowledge/features/feature-x/feature-x.md`            |
| Tuning        | Service Spec + Tech Ref         | `service-my-service.md` + `my-service-optimization.md` |
| Configuration | Standard                        | `knowledge/standards/standard-service-config.md`       |
| Decision      | Feature Hub или несколько specs | Связанные архитектурные документы                      |
| Incident      | Service Spec + Runbook          | `service-x.md` + `troubleshooting-x.md`                |

### Формат ссылок

Используйте reference-style ссылки в footer:

```markdown
## 📚 References

- **Service:** [My Service][my-service-spec]
- **Feature:** [Backup System][backup-feature]
- **Standard:** [Deployment Process][deploy-std]

[my-service-spec]: ../../services/my-service/service-my-service.md
[backup-feature]: ../../features/backup-system/feature-backup-system.md
[deploy-std]: ../standard-feature-specification.md
```

**Не используйте** inline-ссылки `[text](url)` в теле документа.

---

## 📝 Именование Файлов

**Формат:** `YYYY-MM-DD/YYYY-MM-DD-HHMM_<kebab-case-title>.md`

### Компоненты

| Компонент | Формат       | Пример                     | Описание                          |
| --------- | ------------ | -------------------------- | --------------------------------- |
| **Dir**   | `YYYY-MM-DD` | `2026-02-08`               | Директория (дата события)         |
| **Date**  | `YYYY-MM-DD` | `2026-02-08`               | Дата события (в имени файла)      |
| **Time**  | `HHMM`       | `1105`                     | Время (24ч, UTC) без разделителей |
| **Title** | `kebab-case` | `my-service-memory-tuning` | Краткое описание содержимого      |

### Примеры путей

```text
2026-02-08/2026-02-08-1105_my-service-memory-tuning.md
2026-02-02/2026-02-02-0353_service-deployment.md
2026-02-02/2026-02-02-0301_monitoring-tool-installation.md
2026-01-15/2026-01-15-1423_outage-postgres-recovery.md
```

### Правила

- **Никогда** не используйте пробелы — только kebab-case
- **Всегда** создавайте поддиректорию с датой `YYYY-MM-DD`
- **Сокращайте** названия сервисов при известности (`app` vs `application-server`)
- **Указывайте** действие (`tuning`, `deployment`, `rollback`)
- **При повторном событии** — создавайте новый файл с новой датой

---

## ⏱️ Инварианты Времени и Индекса

Для исключения ошибок времени и хронологии используйте следующие инварианты:

1. `Date` в metadata всегда строго `YYYY-MM-DD HH:MM (UTC)`.
2. Timestamp `YYYY-MM-DD-HHMM` в filename является source of truth для:
   - директории `YYYY-MM-DD`;
   - значения `Date` в metadata;
   - позиции записи в `INDEX.md`.
3. Если в файле есть строка `[⬅️ К оглавлению][backlink-index]`, то `Date` располагается строго **после** нее.
4. Ввод `Date` вручную запрещен для новых записей.
5. `INDEX.md` сортируется строго по timestamp (descending).
6. Каждый `memory-YYYY-MM-DD-HHMM` link-id уникален и ссылается на существующий файл.

---

## 🛠️ Стандартный CLI Процесс

Создание новых записей выполняется только через CLI:

```bash
bash scripts/memory/create_agent_memory.sh \
  --title "<h1>" \
  --type <deployment|installation|configuration|tuning|rollout|documentation|incident|decision|rollback> \
  --action "<action>" \
  --status <completed|in_progress|rolled_back> \
  --context-file .tmp/YYYY-MM/YYYY-MM-DD-HHMM_<session>/memory/context.md \
  --changes-file .tmp/YYYY-MM/YYYY-MM-DD-HHMM_<session>/memory/changes.md \
  --validation-file .tmp/YYYY-MM/YYYY-MM-DD-HHMM_<session>/memory/validation.md \
  --impact-file .tmp/YYYY-MM/YYYY-MM-DD-HHMM_<session>/memory/impact.md
```

Гарантии CLI:

- `Date` вычисляется только из `date -u`.
- Генерируются корректные `dir` + `filename`.
- `INDEX.md` пересобирается в правильном порядке автоматически.
- После создания запускается валидация.
- `MUST`: содержательные секции (`Context`, `Changes`, `Validation`, `Impact`) указываются только через `--*-file`.
- `MUST`: inline-аргументы секций не используются.
- `MUST`: файлы для `--*-file` НЕ должны содержать заголовок секции (например, `## 🎯 Context`). Скрипт сам добавляет заголовки в шаблон. Если файл начинается с заголовком секции (`##`), первая строка будет автоматически удалена, но лучше сразу писать только тело секции.
- `MUST`: содержимое `--*-file` оформляется на языке документации проекта (см. раздел «Язык записей»).
- Временные файлы для `--*-file` размещаются в `.tmp/` по стандарту `standard-temp-files-organization`.

Валидация и ремонт:

```bash
bash scripts/memory/validate_agent_memories.sh
bash scripts/memory/validate_agent_memories.sh --fix-index
bash scripts/memory/validate_agent_memories.sh --file knowledge/memory/agent-memories/YYYY-MM-DD/YYYY-MM-DD-HHMM_<title>.md
```

`MUST`: каждый новый или измененный memory-файл должен проходить адресную проверку через `--file` до завершения задачи.

---

## 📊 Архитектура Memory-файлов

```mermaid
flowchart TB
    subgraph Memory["Memory Files"]
        M1[Deployment]
        M2[Installation]
        M3[Tuning]
        M4[Rollout]
        M5[Incident]
        M6[Decision]
    end

    subgraph Docs["Documentation"]
        S[Service Specs]
        F[Feature Hubs]
        T[Tech Reference]
        R[Runbooks]
    end

    M1 --> S
    M3 --> S
    M3 --> T
    M4 --> F
    M4 --> S
    M5 --> S
    M5 --> R
    M6 --> F
    M6 --> S

    Memory -.->|"Context for"| Docs
    Docs -.->|"Referenced by"| Memory
```

**Ключевой принцип:** Memory-файлы дополняют, а не заменяют основную документацию. Они фиксируют историю решений и операционные детали между релизами.

---

## 📚 Шаблоны

### Шаблон Deployment

````markdown
# Service Configuration Deployment

- **Date:** 2026-02-02 03:53 (UTC)
- **Type:** deployment
- **Action:** Deployed updated service configuration
- **Status:** completed

## 🎯 Context

Обновлена конфигурация сервиса для поддержки новых storage volumes.

## 🔧 Changes

- Source: `configs/service-config.yml`
- Script: `scripts/deploy_service.sh`
- Добавлен storage volume `service-data`

## ✅ Validation

```bash
service-cli config validate /etc/service.d
service-cli status  # Проверка запуска
```

## 💥 Impact

- **Сервисы:** Все задания получили доступ к новому volume
- **Требует:** Перезапуск для применения конфигурации

## 📚 References

- **Service:** [My Service][my-service]

[my-service]: ../../services/my-service/service-my-service.md
```

### Шаблон Tuning

```markdown
# Search Service CPU and Thread Tuning

- **Date:** 2026-02-08 11:55 (UTC)
- **Type:** tuning
- **Action:** Оптимизировано распределение CPU и потоков для сервиса speaches
- **Status:** completed

## 🎯 Context

Сервис поиска показывал высокую нагрузку на CPU. Проведён анализ и оптимизация.

## 🔧 Changes

| Параметр | Было  | Стало | Изменение |
| -------- | ----- | ----- | --------- |
| CPU      | 20000 | 28000 | +40%      |
| Threads  | 4     | 28    | +600%     |

```yaml
# В конфигурации ресурсов:
resources:
  cpu: 28000
  memory: 24576
```

## ✅ Validation

- Мониторинг CPU usage в течение 1 часа после деплоя
- Проверка throughput запросов

## 💥 Impact

- **Сервис:** search-service (CPU-bound)
- **Производительность:** Снижение latency на 35%
- **Риски:** Высокое потребление CPU может влиять на другие сервисы

## 📚 References

- **Service:** [Search Service][search-service]
- **Commands:** [Benchmark Commands][benchmark-cmd]

[search-service]: ../../services/search-service/service-search-service.md
[benchmark-cmd]: ./2026-02-08/2026-02-08-2047_search-service-benchmark-commands-documented.md
```

### Шаблон Documentation

```markdown
# Search Service Benchmark Commands Documented

- **Date:** 2026-02-08 20:47 (UTC)
- **Type:** documentation
- **Action:** Задокументированы benchmark-команды для сервиса speaches
- **Status:** completed

## 🎯 Context

Команды для нагрузочного тестирования сервиса поиска были разбросаны. Систематизированы в одном месте.

## 🔧 Commands

### Quick Test

```bash
curl -fsS http://localhost:8080/api/v1/status | jq .
```

### Load Test

```bash
hey -z 30s -c 100 -m POST \
  -H "Content-Type: application/json" \
  -d '{"text":"test"}' \
  http://localhost:8080/api/v1/process
```

## ✅ Validation

- Все команды протестированы на production
- Добавлены в `scripts/benchmark_service.sh`

## 📚 References

- **Service:** [Search Service][search-service]
- **Tuning:** [CPU Tuning][cpu-tuning]

[search-service]: ../../services/search-service/service-search-service.md
[cpu-tuning]: ./2026-02-08/2026-02-08-1155_search-service-cpu-thread-tuning.md
````

---

## ✅ Чеклист

Перед сохранением memory-файла проверьте:

### Обязательные элементы

- [ ] Путь: `YYYY-MM-DD/YYYY-MM-DD-HHMM_kebab-case-title.md`
- [ ] Заголовок H1 без emoji
- [ ] Метаданные: Date, Type, Action, Status
- [ ] Раздел 🎯 Context — почему это сделано
- [ ] Раздел 🔧 Changes — что изменено
- [ ] Раздел ✅ Validation — как проверено
- [ ] Раздел 💥 Impact — что затронуто
- [ ] Раздел 📚 References — если есть связанные документы
- [ ] Footer с reference-ссылками
- [ ] `[⬅️ К оглавлению][backlink-index]` расположен сразу после H1
- [ ] Новая запись создана через `scripts/memory/create_agent_memory.sh`

### Проверки качества

- [ ] Дата и время строго в формате `YYYY-MM-DD HH:MM (UTC)`
- [ ] Action — глагол в прошедшем времени
- [ ] Команды протестированы (можно скопировать и выполнить)
- [ ] Нет inline-ссылок `[text](url)` — только reference-style
- [ ] Все emoji разделов соответствуют стандарту
- [ ] Проверка `markdownlint` пройдена
- [ ] Проверка `bash scripts/memory/validate_agent_memories.sh` пройдена
- [ ] Проверка `bash scripts/memory/validate_agent_memories.sh --file <memory-file-path>` пройдена (MUST)

### Обновление индекса

- [ ] `INDEX.md` обновлен автоматически CLI и валидатором

---

## 🔗 Связь с Другими Стандартами

| Стандарт                                     | Как взаимодействует                                        |
| -------------------------------------------- | ---------------------------------------------------------- |
| [Общий формат спецификаций][standard-common] | Memory-файлы используют reference-ссылки и emoji-заголовки |
| [Спецификация сервисов][standard-service]    | Memory-файлы ссылаются на Service Specs                    |
| [Спецификация фич][standard-feature]         | Memory-файлы ссылаются на Feature Hubs                     |

[standard-common]: ./standard-specification-common-format.md
[standard-service]: ./standard-service-specification.md
[standard-feature]: ./standard-feature-specification.md

---

[backlink-index]: ./INDEX.md
