# AGENTS — verificahub-docs

This repository holds documentation **content** for Verificahub. The Docusaurus
engine lives in `verificahub-docs-site` and pulls this content at build time.

## Purpose
- Keep edits focused on Markdown in `docs/` and the `registry/links.json` map.
- Russian (`ru`) is the default and only locale unless a task adds translations.
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

## Standard flow
1. Apply the smallest valid change for the task.
2. Verify links/paths when touching navigation pages (`index.md`, categories).
3. Update `registry/links.json` if routes changed.
4. Commit with a clear, scoped message; push to `production` to deploy.

## AI context
Shareable context lives in `ai/` (tracked). Ephemeral cache goes in `.ai/`
(git-ignored).
