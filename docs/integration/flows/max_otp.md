---
title: max_otp — код в мессенджере MAX
sidebar_label: max_otp
sidebar_position: 7
---

# `max_otp` — код в мессенджере MAX

Код приходит пользователю сообщением в MAX. Никаких ссылок и дополнительных действий: сообщение уже
доставлено к моменту, когда вы получаете ответ на `POST /v1/verify`.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    participant MAX as MAX
    App->>VH: POST /v1/verify {method: max_otp}
    VH->>MAX: доставка кода
    MAX-->>U: код в MAX
    VH-->>App: request_id, code_length, status: sent
    App-->>U: поле ввода кода (code_length цифр)
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (или 400 invalid_code + attempts_remaining)
```

## Что учесть

- **Не путайте с [`max_bot`](./max_bot.md).** Там пользователь должен сам открыть бота и поделиться
  номером; здесь — ничего не должен, сообщение просто приходит. Большинству сценариев нужен именно
  `max_otp`.
- **Показывайте поле ввода сразу** после ответа на `POST /v1/verify`: код уже отправлен.
- **У пользователя должен быть MAX на этом номере** — иначе доставлять некуда.
