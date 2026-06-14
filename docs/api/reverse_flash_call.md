---
title: reverse_flash_call
sidebar_label: reverse_flash_call
sidebar_position: 2
---

# Метод `reverse_flash_call`

Пользователь сам звонит на наш номер из пула `+7`. Звонок не принимается — номер
подтверждается по Caller ID. Самый быстрый и дешёвый способ — от 0,25 ₽ за
подтверждение.

## Запрос

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -H "Authorization: Bearer sk_live_••••" \
  -d '{ "phone": "+79009009090", "method": "reverse_flash_call" }'
```

## Ответ

```json
{
  "id": "vf_5f3a",
  "status": "sent",
  "number_to_call": "+79009009090",
  "cost": { "amount": 0.25, "currency": "RUB" }
}
```

Покажите пользователю `number_to_call` и попросите позвонить. Результат
подтверждения приходит на ваш webhook.
