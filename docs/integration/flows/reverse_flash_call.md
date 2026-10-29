---
title: reverse_flash_call — подтверждение без ввода кода
sidebar_label: reverse_flash_call
sidebar_position: 2
---

# `reverse_flash_call` — без ввода кода

Пользователь сам звонит на выданный номер, мы подтверждаем по Caller ID. Код никто не вводит.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: reverse_flash_call}
    VH-->>App: request_id, number_to_call, status: sent
    App-->>U: «Позвоните на number_to_call»
    U->>VH: звонок-сброс на этот номер
    VH->>VH: сверка Caller ID → verified
    VH-->>App: webhook verification.verified
    App->>VH: GET /v1/verify/{request_id} (если опрашиваете)
    VH-->>App: status: verified
```

Покажите пользователю `number_to_call`, а сами ждите `verified` (по вебхуку или опросом).

## Что учесть

- **`/v1/verify/check` для этого метода не используется.** Вводить нечего.
- **Номер выдаётся на сессию.** Берите его из ответа каждый раз, не кэшируйте: в следующей сессии он
  может быть другим.
- **Скрытый Caller ID не подтвердится.** Если пользователь звонит с антиопределителем, сопоставлять
  нечего, и сессия дойдёт до `expired`.
