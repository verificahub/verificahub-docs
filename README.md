# Verificahub Docs

Источник истины для документации Verificahub (контент в Markdown).

Работает в паре с репозиторием движка [`verificahub-docs-site`](https://github.com/verificahub/verificahub-docs-site):
тот при сборке клонирует этот репозиторий, подставляет `docs/`, собирает
Docusaurus-сайт и публикует его на GitHub Pages (https://docs.verificahub.ru).

## Структура

```text
verificahub-docs/
├─ docs/                     # контент (локаль ru)
│  ├─ index.md
│  ├─ start/                 # быстрый старт
│  ├─ api/                   # методы API
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
