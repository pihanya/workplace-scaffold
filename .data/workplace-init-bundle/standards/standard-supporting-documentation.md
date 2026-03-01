# 📋 Стандарт вспомогательной документации

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: standard-supporting-documentation
depends_on:
  - knowledge/standards/standard-specification-common-format.md
  - knowledge/standards/standard-service-specification.md
provides_for:
  - knowledge/services/*/docs/TECHNOLOGY.md
  - knowledge/services/*/docs/RUNBOOK.md
  - knowledge/services/*/docs/TROUBLESHOOTING.md
-->

Этот стандарт определяет структуру и правила ведения вспомогательной документации для сервисов в `knowledge/services/<service-name>/docs/`.

Цель стандарта:

- отделить спецификацию возможностей сервиса (что умеет) от operational/implementation деталей (как устроено и как поддерживать);
- унифицировать формат эксплуатационной документации для разработчиков, SRE и on-call поддержки;
- обеспечить быстрый онбординг и снижение времени диагностики инцидентов.

---

## 📂 Структура и размещение

Вспомогательная документация хранится в директории:

`knowledge/services/<service-name>/docs/`

Рекомендуемая структура:

```text
knowledge/
└── services/
    └── <service-name>/
        ├── service-<service-name>.md
        ├── features/
        │   └── <feature-name>/
        │       └── feature-<feature-name>.md
        └── docs/
            ├── TECHNOLOGY.md
            ├── RUNBOOK.md
            └── TROUBLESHOOTING.md
```

### Когда создавать документы

| Документ             | Обязательность | Когда нужен                                                                                                      |
| :------------------- | :------------- | :--------------------------------------------------------------------------------------------------------------- |
| `TECHNOLOGY.md`      | Обязателен     | Сервис содержит интеграции, нетривиальную конфигурацию или внутренние механики (кэш, ретраи, очереди, scheduler) |
| `RUNBOOK.md`         | Обязателен     | Сервис деплоится в среду, где есть эксплуатационные процедуры (start/stop/deploy/rollback/backup/restore)        |
| `TROUBLESHOOTING.md` | Обязателен     | Для сервиса возможны инциденты, деградация или зависимость от внешних систем                                     |

---

## 📖 Technology Reference (`TECHNOLOGY.md`)

### Область ответственности: Technology Reference

Документ отвечает на вопрос: **как сервис работает внутри**.

`TECHNOLOGY.md` ДОЛЖЕН содержать:

- внутреннюю архитектуру и ключевые технические решения;
- ключевые параметры конфигурации и их влияние;
- зависимости на инфраструктуру и внешние сервисы;
- правила мониторинга и диагностики на уровне технологий.

`TECHNOLOGY.md` НЕ ДОЛЖЕН содержать:

- пошаговые процедуры инцидент-реакции (это `TROUBLESHOOTING.md`);
- операции backup/restore и регламентные операции (это `RUNBOOK.md`);
- бизнес-обоснование фичи (это Feature Hub/Node документы).

### Шаблон

````markdown
# 📖 <Technology Name> — карточка технологии

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: tech-<service-name>
depends_on:
  - knowledge/services/<service-name>/service-<service-name>.md
  - knowledge/standards/standard-supporting-documentation.md
provides_for: []
-->

## 🎯 Предназначение

Краткое описание роли технологии в сервисе.

## 🏗️ Архитектура

Описание ключевых компонентов, потоков данных и ограничений.

## 🔌 Порты и сетевые зависимости

Точки входа/выхода, протоколы, внешние хосты/сервисы.

## 💾 Данные и persistence

Типы хранилищ, модели данных, политика хранения.

## 🧮 Ресурсы

CPU/RAM/IO требования, baseline и пиковые значения.

## 🐳 Local Development

Как поднять технологию локально (Docker/Compose/Standalone).

## ⚙️ Ключевые параметры (Deep Dive)

### Parameter Group 1

- `PARAM_A`: что задаёт, допустимые диапазоны, дефолт
- `PARAM_B`: влияние на latency/throughput/reliability

## 🛡️ Безопасность и hardening

Базовые требования безопасности и секреты/доступы.

## 🔍 Операционные проверки

```bash
# Health check
curl -fsS http://<host>:<port>/health
```

## 📚 References

[backlink-index]: ../service-<service-name>.md
````

---

## 🧯 Operational Runbook (`RUNBOOK.md`)

### Область ответственности: Operational Runbook

Документ отвечает на вопрос: **как безопасно выполнять регулярные и аварийные операции**.

`RUNBOOK.md` ДОЛЖЕН содержать:

- процедуры deploy/restart/rollback;
- процедуры backup/restore для stateful компонентов;
- критерии успешности/неуспешности операции;
- pre-check и post-check шаги.

### Шаблон Runbook

````markdown
# 🧯 <Service Name> Operations

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: runbook-<service-name>
depends_on:
  - knowledge/services/<service-name>/service-<service-name>.md
  - knowledge/standards/standard-supporting-documentation.md
provides_for: []
-->

## 📘 Overview

Краткое описание назначения runbook и зоны ответственности.

## 🚀 Deploy / Update

### Prerequisites

- доступ к среде деплоя
- актуальный артефакт/образ

### Процедура

```bash
# Option A: Nomad
nomad job validate projects/<service>/<service>.nomad.hcl
nomad job run projects/<service>/<service>.nomad.hcl

# Option B: Docker Compose
docker compose -f projects/<service>/docker-compose.yml up -d

# Option C: Kubernetes
kubectl apply -f projects/<service>/deployment.yaml
kubectl rollout status deployment/<service>
```

### Post-check

- сервис в состоянии running/healthy
- ключевой health endpoint отвечает

## 🧯 Rollback

Пошаговая процедура отката к предыдущей стабильной версии.

## 💾 Backup

Периодичность backup, формат и место хранения.

## 🔄 Restore

Пошаговая процедура восстановления и верификация целостности.

## ✅ Validation

Чеклист после операционных действий:

- [ ] health check успешен
- [ ] критичные бизнес-сценарии работают
- [ ] ошибки в логах отсутствуют/в допустимом диапазоне

## 📚 References

[backlink-index]: ../service-<service-name>.md
````

---

## 🔍 Troubleshooting Guide (`TROUBLESHOOTING.md`)

### Область ответственности: Troubleshooting Guide

Документ отвечает на вопрос: **как диагностировать и устранять типовые проблемы**.

`TROUBLESHOOTING.md` ДОЛЖЕН содержать:

- перечень симптомов и вероятных причин;
- диагностические команды;
- проверенные шаги устранения;
- критерии эскалации.

### Шаблон Troubleshooting

````markdown
# 🔍 <Service Name> Troubleshooting

[⬅️ К оглавлению][backlink-index]

<!-- doc-deps
id: troubleshooting-<service-name>
depends_on:
  - knowledge/services/<service-name>/service-<service-name>.md
  - knowledge/standards/standard-supporting-documentation.md
provides_for: []
-->

## 📘 Overview

Описание зоны покрытия и границ troubleshooting-гайда.

## 🔧 Диагностические команды

```bash
# Service status (выбрать релевантный вариант)
nomad job status <service>
docker compose -f projects/<service>/docker-compose.yml ps
kubectl get pods -n <namespace>

# Logs
nomad alloc logs -stderr -tail -n 200 -job <service>
docker compose -f projects/<service>/docker-compose.yml logs --tail=200 <service>
kubectl logs deployment/<service> --tail=200 -n <namespace>

# Health check
curl -fsS http://<host>:<port>/health
```

## 🚨 Известные проблемы

### Проблема 1: <Краткое описание>

- **Симптомы:** ...
- **Причина:** ...
- **Решение:** ...

### Проблема 2: <Краткое описание>

- **Симптомы:** ...
- **Причина:** ...
- **Решение:** ...

## 📊 Мониторинг

- ключевые метрики
- критичные логи/паттерны ошибок
- warning signs перед деградацией

## 🆘 Emergency Procedures

- Service Down
- Data Corruption
- Upstream Dependency Failure

## 📚 References

[backlink-index]: ../service-<service-name>.md
````

---

## 📝 Правила оформления

1. Все документы в `docs/` следуют [Стандарту общего формата][standard-common-format].
2. Используются только reference-style ссылки (`[label][ref-id]` + footer).
3. Команды указываются с понятным execution context (из корня или с явным `cd`).
4. Диаграммы оформляются через Mermaid.
5. В `service-<service-name>.md` должны быть ссылки на `TECHNOLOGY.md`, `RUNBOOK.md` и `TROUBLESHOOTING.md`.

---

## ✅ Чеклист

### Обязательные элементы

- [ ] Для сервиса создан `docs/TECHNOLOGY.md`
- [ ] Для сервиса создан `docs/RUNBOOK.md`
- [ ] Для сервиса создан `docs/TROUBLESHOOTING.md`
- [ ] Документы связаны ссылками со спецификацией сервиса
- [ ] Заполнены ссылки `References` и корректен backlink

### Проверки качества

- [ ] Нет дублирования между `service-*.md` и `docs/*`
- [ ] Процедуры в runbook воспроизводимы
- [ ] Troubleshooting покрывает топ-риски сервиса

---

## 🔗 Связь с другими стандартами

- [Стандарт общего формата спецификаций][standard-common-format]
- [Стандарт специфицирования сервисов][standard-service-spec]
- [Стандарт управления зависимостями между сервисами][standard-service-deps]

[standard-common-format]: ./standard-specification-common-format.md
[standard-service-spec]: ./standard-service-specification.md
[standard-service-deps]: ./standard-service-dependencies.md
[backlink-index]: ../INDEX.md
