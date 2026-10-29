---
title: max_bot — код через бота в MAX
sidebar_label: max_bot
sidebar_position: 8
---

# `max_bot` — код через бота в MAX

Отличие от [`max_otp`](./max_otp.md) — пользователь должен сам начать диалог. В ответ на
`POST /v1/verify` приходит поле `deep_link` (`https://max.ru/<bot>?start=<token>`); покажите его
ссылкой или QR-кодом. Пользователь открывает бота, жмёт **«Поделиться номером»** — и только после
этого код приходит в MAX.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    participant MAX as MAX
    App->>VH: POST /v1/verify {method: max_bot}
    VH-->>App: request_id, deep_link, code_length, status: sent
    App-->>U: ссылка или QR на deep_link
    U->>MAX: открывает бота, жмёт «Поделиться номером»
    MAX->>VH: контакт пользователя
    VH->>MAX: доставка кода → delivered
    MAX-->>U: код в MAX
    App-->>U: поле ввода кода (code_length цифр)
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (или 400 invalid_code + attempts_remaining)
```

## Что учесть

- **Номер должен совпадать.** Пользователь обязан поделиться **тем же** номером, для которого
  запущена проверка, иначе код не отправляется.
- **Статусы различают шаги.** До «Поделиться номером» — `sent`; после доставки — `delivered` и
  `awaiting_code: true`.
- **Списание — по факту доставки.** Если пользователь не открыл бота или не поделился номером,
  оплата не взимается.
- **Лишний шаг стоит конверсии.** Если вам не нужен именно диалог с ботом, берите
  [`max_otp`](./max_otp.md).
