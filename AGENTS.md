# AGENTS — verificahub-docs

This repository holds documentation **content** for Verificahub. The Docusaurus
engine lives in `verificahub-docs-site` and pulls this content at build time.

## Purpose
- Keep edits focused on Markdown in `docs/` and the `registry/links.json` map.
- Russian (`ru`) is the default locale and lives in `docs/`. English (`en`)
  translations live in `i18n/en/docusaurus-plugin-content-docs/current/` and
  mirror `docs/` paths/filenames; legal docs are intentionally RU-only (see
  `i18n/README.md`). When you change a `docs/` page, update its `i18n/en/`
  counterpart (if one exists), or note it's now out of sync.
- Keep changes small, reviewable, and safe for production deploys.

## Working rules
- Prefer direct Markdown edits over structural churn.
- Keep routes and slugs stable unless the task explicitly requires changes.
- **Keep `registry/links.json` in sync** with any docs structure change (new,
  removed, renamed, or re-routed pages) — especially legal docs.
- Each section has a `_category_.json` controlling its sidebar label/position.
- Legal docs are sokращённые placeholders — mark substantive legal text as
  draft until reviewed; don't invent binding terms.

## Brand
- Tone matches the landing (`verificahub-web-landing`): plain Russian, concrete.
- Product facts: pay-as-you-go, обратный flash-call от 0,25 ₽; API base
  `https://api.verificahub.ru/v1`; site `https://verificahub.ru`.

## Справочник API и его перевод

`openapi/verificahub-api-v1.json` — спецификация, как её экспортирует бэкенд. **Править её
руками нельзя**: следующий экспорт затрёт правки. Она на английском и остаётся источником истины.

Русская версия справочника собирается наложением оверлея `openapi/ru-overlay.json`:

```json
{ "/paths/~1v1~1verify/post/summary": {
    "en": "Initiate a phone verification.",
    "ru": "Инициировать подтверждение номера." } }
```

Ключ — JSON Pointer (RFC 6901) до переводимой строки. Рядом с переводом хранится английский
оригинал **на момент перевода**: если в спецификации он изменился, перевод считается устаревшим
и в сборке показывается английский текст, а не расходящийся с оригиналом русский. Без этой
сверки переводы тихо протухают — это главная причина, по которой переводы документации
перестают соответствовать API.

После экспорта новой спецификации:

```bash
cd ../verificahub-docs-site
npm run api:i18n              # что не переведено, что устарело, что осиротело
npm run api:i18n -- --write   # дописать заготовки, затем заполнить поля "ru"
```

Страницы справочника (`docs/reference/`, `i18n/en/.../reference/`) генерируются на сборке
и в репозиториях не хранятся. Русская версия идёт в `docs/` (локаль по умолчанию),
английская — в `i18n/en/`.

## Generated pages — do not edit here

Both changelogs are written by CI on every production deploy of their source repo.
Edits made in this repo are overwritten on the next one — change the source instead.

| Page (RU + EN) | Written from |
| --- | --- |
| `docs/changelog/api.md` | `verificahub-api/docs/changelog/{ru,en}.md` |
| `docs/changelog/app.md` | `verificahub-web-app/docs/changelog/{ru,en}.md` |

Each page pins its public URL with `slug:` in its front matter, so the files can be
regrouped without breaking links. `_category_.json` is hand-maintained — CI writes only
the two `.md` files and never touches it.

The two jobs push to this same branch and each copies its files wholesale, so they
must never be pointed at the same page.

## Standard flow
1. Apply the smallest valid change for the task.
2. Verify links/paths when touching navigation pages (`index.md`, categories).
3. Update `registry/links.json` if routes changed.
4. Commit with a clear, scoped message; push to `production` to deploy.

## AI context
Shareable context lives in `ai/` (tracked). Ephemeral cache goes in `.ai/`
(git-ignored).
