---
title: Подтверждение по коду (Telegram, SMS, Flash Call)
sidebar_label: Подтверждение по коду
sidebar_position: 3
---

# Подтверждение по коду

Методы `telegram`, `sms` и `flash_call` работают по одному сценарию: пользователь получает или считывает короткий код, вводит его в вашем приложении, а вы подтверждаете код через API. Отличается только канал доставки.

| Метод | Откуда пользователь берёт код |
| --- | --- |
| `telegram` | Код приходит сообщением в Telegram |
| `sms` | Код приходит в SMS |
| `flash_call` | Звонок-сброс на номер пользователя; код — **последние цифры входящего номера** |

## Шаг 1. Инициировать проверку

Укажите нужный `method` (`sms`, `telegram` или `flash_call`):

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{
    "phone_number": "+79991234567",
    "method": "sms"
  }'
```

Ответ `201 Created`:

```json
{
  "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "phone_number": "+79991234567",
  "method": "sms",
  "status": "sent",
  "cost": { "amount": 3.0, "currency": "RUB" },
  "expires_at": "2026-06-17T12:35:00Z",
  "code_length": 4
}
```

`code_length` — сколько цифр будет в коде. Используйте его, чтобы настроить поле ввода.

## Шаг 2. Пользователь вводит код

- **Telegram / SMS** — пользователь получает код в сообщении и вводит его.
- **Flash Call** — пользователю поступает короткий звонок-сброс; код — это последние `code_length` цифр номера, с которого пришёл звонок. Пользователь вводит их (брать трубку не нужно).

## Шаг 3. Проверить код

Отправьте введённый код вместе с `request_id`:

```bash
curl -X POST https://api.verificahub.ru/v1/verify/check \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{
    "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "code": "1234"
  }'
```

Ответ `200 OK`:

```json
{
  "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "status": "verified",
  "phone_number": "+79991234567"
}
```

`status: "verified"` — код верный, номер подтверждён. Если код неверный или сессия истекла, вернётся `400` — покажите пользователю сообщение и предложите запросить код заново.

:::note
Для метода `reverse_flash_call` этот сценарий не применяется — там подтверждение автоматическое, без ввода кода. См. [Обратный flash-call](./reverse_flash_call.md).
:::
