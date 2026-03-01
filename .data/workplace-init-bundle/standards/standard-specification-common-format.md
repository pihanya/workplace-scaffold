# 📋 Стандарт Общего Формата Спецификаций

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: standard-specification-common-format
depends_on: []
provides_for:
  - knowledge/standards/standard-service-specification.md
  - knowledge/standards/standard-feature-specification.md
  - knowledge/standards/standard-service-dependencies.md
  - knowledge/standards/standard-supporting-documentation.md
-->

Этот стандарт определяет универсальный формат для документов спецификаций (Markdown) в проекте. Соблюдение этого формата обеспечивает:

- **Отслеживаемость**: Зависимости между документами (`doc-deps`) машиночитаемы.
- **Консистентность**: Единообразная структура (заголовок, обратные ссылки, референсы) упрощает чтение и навигацию.
- **Поддержка инструментов**: Парсеры и линтеры могут валидировать и визуализировать граф документации.

---

## 📂 Структура Файла

Каждый файл спецификации ДОЛЖЕН следовать этой структуре:

| Порядок | Раздел       | Описание                               |
| :------ | :----------- | :------------------------------------- |
| 1       | **Header**   | H1 заголовок с emoji + обратная ссылка |
| 2       | **Metadata** | Блок комментария `doc-deps` (YAML)     |
| 3       | **Body**     | Основное содержимое спецификации       |
| 4       | **Footer**   | Определение reference-ссылок           |

---

## 📘 Требования к Структуре

### Заголовок (Header)

**Формат**: `# <Emoji> <Title>`

**Правила:**

- Первая строка должна быть H1 заголовком, начинающимся с релевантного emoji, за которым следует пробел и название.
- Emoji должно отражать суть документа (например, 🚀 для фич, 📋 для спецификаций, 🛠️ для технических документов).

**Пример**: `# 🚀 Feature Specification`

---

### Обратная Ссылка (Backlink)

**Расположение**: Сразу после заголовка.

**Формат**: `[⬅️ К оглавлению][backlink-index]`

**Назначение**: Обеспечивает быструю навигацию к родительскому индексу.

**Правила:**

- Использовать reference-style ссылки.
- Определение ссылки должно быть в footer-секции.

---

### Метаданные Зависимостей (`doc-deps`)

**Расположение**: Сразу после обратной ссылки.

**Формат**: HTML-комментарий с YAML-блоком.

**Поля:**

| Поле           | Тип          | Описание                                                         | Пример               |
| :------------- | :----------- | :--------------------------------------------------------------- | :------------------- |
| `id`           | string       | Уникальный идентификатор документа (kebab-case)                  | `my-spec-id`         |
| `depends_on`   | list[string] | Upstream зависимости (файлы, от которых зависит этот документ)   | `[docs/overview.md]` |
| `provides_for` | list[string] | Downstream документы (файлы, которые зависят от этого документа) | `[docs/prd.md]`      |

**Пути**: Пути должны быть указаны относительно корня репозитория или корня модуля (например, `knowledge/...` или `docs/...` в зависимости от контекста).

**Пример блока:**

```yaml
<!-- doc-deps
id: my-spec-id
depends_on:
  - docs/concept/product-overview.md
provides_for:
  - docs/product-requirements/02-prd-traceability.md
-->
```

---

### Reference-ссылки (Footer)

**Правила:**

- **ВСЕ Markdown-ссылки** (внутренние на другие документы и внешние URL) ДОЛЖНЫ быть оформлены как reference-style: `[Текст ссылки][link-id]`.
- Использование inline-ссылок `[Text](url)` **ЗАПРЕЩЕНО** в теле документа.
- Все ссылки определяются в секции Footer в конце файла.
- Якорь `[backlink-index]` **ВСЕГДА** должен быть последней строкой в файле.

**Формат**: `[link-id]: https://url.com "Optional Title"`

**Пример:**

```markdown
...
См. также [Feature Hub][feature-hub].

[feature-hub]: ../../features/feature-name/feature-feature-name.md
[backlink-index]: index.md
```

---

## 📋 Шаблон Спецификации

Ниже приведен шаблон для нового файла спецификации:

````markdown
# 📋 Specification Name

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: spec-feature-name
depends_on:
  - docs/source-doc.md
provides_for: []
-->

## 📘 Цель

Краткое описание цели и назначения документа.

## 🧠 Функциональность

Детальное описание функциональности или архитектуры.

### Подраздел 1

Описание компонента или логики.

### Подраздел 2

Описание компонента или логики.

---

## 🌐 API / Интеграции

*(Если применимо)*

Описание API-контрактов или интеграций с внешними сервисами.

| Endpoint        | Method | Описание          |
| :-------------- | :----- | :---------------- |
| `/api/resource` | POST   | Создание ресурса  |
| `/api/resource` | GET    | Получение ресурса |

---

## 📚 References

### Внутренняя документация

- **Feature Hub**: Ссылка на hub-документ фичи в `knowledge/features/`
- **Service Spec**: Ссылка на спецификацию сервиса
- **Related Standards**: Ссылки на связанные стандарты

### Внешние ресурсы

- **Official Documentation**: [External Service][service-link]
- **GitHub Repository**: [Repository Link][repo-link]

[service-link]: https://service.com "Service Name"
[repo-link]: https://github.com/org/repo
[backlink-index]: index.md
````

---

## 📝 Правила Оформления

### Общие требования

1. **Язык**: Основной язык документации — русский. Технические термины могут оставаться на английском (например, API, endpoint, service).
2. **Emoji**: Использовать emoji в заголовках H2 и H3 для визуального разделения секций.
3. **Структура**: Логическое деление на секции с четкими заголовками.
4. **Таблицы**: Активно использовать таблицы для структурированной информации.
5. **Диаграммы**: Для схем и графов в документации использовать Mermaid; ASCII-диаграммы не использовать.

### Mapping emoji для типов документов (H1)

Каждый тип документа должен использовать специфичный emoji в H1 заголовке:

| Тип документа        | Emoji | Формат H1                                  | Область ответственности                              |
| :------------------- | :---- | :----------------------------------------- | :--------------------------------------------------- |
| Service Spec         | 🛠️     | `# 🛠️ <Service Name> Service Specification` | **WHAT**: Возможности сервиса, API, зависимости      |
| Feature Hub          | 🚀     | `# 🚀 <Feature Name>`                       | **WHAT & WHY**: Бизнес-задача, архитектурные решения |
| Feature Node         | 🔧     | `# 🔧 <Feature Name> - <Service Name>`      | **HOW (service)**: Реализация фичи в сервисе         |
| Standard             | 📋     | `# 📋 Стандарт <Name>`                      | Правила и шаблоны для документации                   |
| Technology Reference | 📖     | `# 📖 <Technology Name>`                    | **HOW (tech)**: Внутренняя работа технологии         |
| Operational Runbook  | 🧯     | `# 🧯 <Service Name> Operations`            | Пошаговые операционные процедуры                     |
| Troubleshooting      | 🔍     | `# 🔍 <Service Name> Troubleshooting`       | Диагностика и решение проблем                        |

### Принцип "What vs How"

Документация разделяется по уровню абстракции:

| Уровень            | Вопрос               | Типы документов            | Аудитория                 |
| :----------------- | :------------------- | :------------------------- | :------------------------ |
| **Business**       | Зачем? Что даёт?     | Feature Hub                | Stakeholders, архитекторы |
| **Capabilities**   | Что умеет?           | Service Spec, Feature Node | Разработчики, DevOps      |
| **Implementation** | Как работает внутри? | Technology Reference       | DevOps, SRE               |
| **Operations**     | Как обслуживать?     | Runbook, Troubleshooting   | SRE, Operations           |

**Правило:** Каждый документ отвечает на вопросы СВОЕГО уровня. Для деталей других уровней — ссылка на соответствующий документ.

### Рекомендованные emoji для заголовков разделов (H2/H3)

| Emoji | Назначение                              | Пример раздела             |
| :---- | :-------------------------------------- | :------------------------- |
| 📘     | Цель, описание, введение                | `## 📘 Цель`                |
| 🧠     | Функциональность, архитектура, логика   | `## 🧠 Функциональность`    |
| 🌐     | API, интеграции, интерфейсы             | `## 🌐 API Интерфейсы`      |
| 🔧     | Конфигурация, развертывание             | `## 🔧 Deployment`          |
| 📊     | Метрики, мониторинг, производительность | `## 📊 Monitoring`          |
| 🧪     | Тестирование, валидация                 | `## 🧪 Validation Tests`    |
| ⚠️     | Предупреждения, ошибки, ограничения     | `## ⚠️ Обработка ошибок`    |
| 📚     | Ссылки, референсы, связанные документы  | `## 📚 References`          |
| 📂     | Структура директорий, файлов            | `## 📂 Структура`           |
| 🔗     | Зависимости, связи между компонентами   | `## 🔗 Зависимости`         |
| 🚀     | Развертывание, deployment sequence      | `## 🚀 Deployment Sequence` |
| 💥     | Blast radius, влияние изменений         | `## 💥 Blast Radius`        |

### Стиль написания

- **Тон**: Профессиональный, формальный, с использованием корпоративной терминологии.
- **Абзацы**: Разбивайте текст на смысловые абзацы для удобства чтения.
- **Списки**: Используйте списки для перечислений и чек-листов.
- **Код-блоки**: Используйте код-блоки для примеров конфигураций, команд, API-запросов.

---

## 📚 Примеры Применения

### Пример 1: Service Specification

Спецификации сервисов (`knowledge/services/<name>/service-<name>.md`) следуют этому стандарту:

```text
# Payment Service Specification

## 📘 Цель

Краткое описание сервиса обработки платежей.

## 🧠 Функциональность / Архитектура

Описание компонентов и архитектуры.

## 🌐 API Интерфейсы

Описание API endpoints.

## 📚 References

Ссылки на hub-документы фич и deployment config.
```

### Пример 2: Feature Hub

Hub-документы фич (`knowledge/features/<feature>/feature-<feature>.md`) также следуют стандарту:

```text
# User Authentication

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: feature-user-authentication
depends_on: []
provides_for:
  - ../../services/auth/features/user-authentication/feature-user-authentication.md
-->

## 📘 Общее описание задачи

Описание функциональности аутентификации пользователей.

## 🏗️ Архитектурные принципы

Паттерны и подходы к реализации.

## 🚀 Deployment Sequence

Порядок развертывания компонентов.

## 💥 Blast Radius

Оценка влияния изменений.

[backlink-index]: ../../index.md
```

### Пример 3: Feature Node

Node-документы фич (`knowledge/services/<service>/features/<feature>/feature-<feature>.md`):

```text
# User Authentication - Auth Service

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: service-auth-feature-user-authentication
depends_on:
  - ../../../../features/user-authentication/feature-user-authentication.md
provides_for: []
-->

## 📘 Цель

Роль сервиса Auth в реализации функциональности аутентификации.

## 🧠 Функциональность / Архитектура

Детальное описание логики работы сервиса.

## 🌐 Интеграция с API

Описание API контрактов.

[backlink-index]: ../../../index.md
```

---

## 🔗 Связь с Другими Стандартами

| Стандарт                                                 | Как взаимодействует                                                           |
| :------------------------------------------------------- | :---------------------------------------------------------------------------- |
| [Service Specification Standard][standard-services]      | Спецификации сервисов следуют этому общему формату для структуры и референсов |
| [Feature Specification Standard][standard-features]      | Feature Hub и Node документы следуют этому формату для метаданных и навигации |
| [Service Dependencies Standard][standard-deps]           | Использует reference-ссылки для связывания документов зависимостей            |
| [Supporting Documentation Standard][standard-supporting] | Шаблоны для Technology Reference, Runbook, Troubleshooting                    |

---

## 📝 Правила Использования Стандарта

1. **Обязательные элементы**: Все спецификации должны включать header, backlink, `doc-deps` блок и footer с reference-ссылками.
2. **Консистентность ID**: Идентификаторы `id` в блоке `doc-deps` должны быть уникальными в рамках проекта.
3. **Актуализация зависимостей**: При изменении структуры документов обновляйте поля `depends_on` и `provides_for`.
4. **Навигация**: Используйте reference-ссылки для связывания документов (не inline-ссылки).
5. **Emoji**: Используйте emoji в заголовках H1 для визуальной идентификации типа документа.

---

## 📋 Checklist для Нового Документа

При создании новой спецификации:

- [ ] H1 заголовок с emoji и названием
- [ ] Обратная ссылка `[⬅️ К оглавлению][backlink-index]`
- [ ] Блок `doc-deps` с уникальным `id` и актуальными зависимостями
- [ ] Основное содержимое разбито на логические секции с emoji-заголовками
- [ ] Footer с определением **всех** reference-ссылок (inline ссылки запрещены)
- [ ] `[backlink-index]` является последней строкой файла
- [ ] Ссылки на связанные документы (feature hubs, service specs, standards)
- [ ] Актуализированы `depends_on` и `provides_for` в связанных документах

---

[standard-services]: ./standard-service-specification.md
[standard-features]: ./standard-feature-specification.md
[standard-deps]: ./standard-service-dependencies.md
[standard-supporting]: ./standard-supporting-documentation.md
[backlink-index]: ./INDEX.md
