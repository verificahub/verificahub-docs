# Verificahub Docs

Источник истины для документации Verificahub (контент в Markdown).

Работает в паре с репозиторием движка [`verificahub-docs-site`](https://github.com/verificahub/verificahub-docs-site):
тот при сборке клонирует этот репозиторий, подставляет `docs/`, собирает
Docusaurus-сайт и публикует его в контейнере на VPS (https://docs.verificahub.ru).

## Структура

```text
verificahub-docs/
├─ docs/                     # контент (локаль ru)
│  ├─ index.md
│  ├─ start/                 # быстрый старт
│  ├─ integration/           # интеграционные сценарии
│  ├─ guides/                # руководства, включая аккаунты MAX
│  ├─ reference/             # генерируется движком из OpenAPI
│  └─ legal/                 # оферта, политика, условия
├─ registry/links.json       # карта ссылок для внешних потребителей
└─ .github/workflows/trigger.yml   # дёргает пересборку сайта
```

## Поток работы

1. Правите Markdown в `docs/`.
2. Синхронизируете `registry/links.json` при изменении структуры/маршрутов.
3. Пуш в ветку `production` → `trigger.yml` отправляет `repository_dispatch`
   (`docs-update`) в `verificahub-docs-site`, и сайт пересобирается.

## Настройка

В этом репозитории нужен секрет:
- `REPO_TOKEN` — токен с правом запускать `repository_dispatch` в
  `verificahub-docs-site`.

Соглашения для людей и AI-агентов — в [AGENTS.md](AGENTS.md).

## Актуальность справочника API

`openapi/verificahub-api-v1.json` экспортируется из текущего backend Swagger;
не правьте его вручную. `openapi/export-metadata.json` фиксирует источник последнего
экспорта. Публичный справочник включает только внешний API; маршруты кабинета MAX
остаются в отдельной dashboard-спецификации в репозиториях API и web-app.
После успешного API-деплоя CI автоматически публикует свежую внешнюю спецификацию
и её метаданные вместе с changelog. Dashboard/internal документы в публичный сайт не попадают.
Для локального экспорта без запущенного API и БД выполните в репозитории API:
`./scripts/regen-openapi.sh` — скрипт использует тестовый хост.
После изменения контракта обновляйте переводы и примеры, затем собирайте RU и EN версии сайта.
Юридические изменения проходят отдельную проверку и не публикуются как технические правки.
