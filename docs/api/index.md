---
title: Обзор API
sidebar_label: Обзор API
sidebar_position: 1
---

# API Verificahub

Единый REST-эндпоинт для всех методов верификации. Метод выбирается полем
`method` в теле запроса.

| Метод                | Значение `method`     | Статус   |
| -------------------- | --------------------- | -------- |
| Обратный flash-call  | `reverse_flash_call`  | Доступен |
| Flash-call           | `flash_call`          | Скоро    |
| Голосовой код        | `voice`               | Скоро    |
| SMS                  | `sms`                 | Скоро    |

- Базовый URL: `https://api.verificahub.ru/v1`
- Авторизация: заголовок `Authorization: Bearer <API-ключ>`
- Формат: `application/json`

Доступные методы:

- [reverse_flash_call](./reverse_flash_call)
