---
title: sms — код в SMS
sidebar_label: sms
sidebar_position: 3
---

# `sms` — код в SMS

Мы генерируем код и отправляем его пользователю по SMS; пользователь вводит код у вас, вы отправляете
его на проверку.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: sms}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: SMS с кодом
    App-->>U: поле ввода кода (code_length цифр)
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (или 400 invalid_code + attempts_remaining)
```

## Что учесть

- **Длина кода — в `code_length`.** Не зашивайте её в вёрстку: она задаётся настройкой платформы.
- **При неверном коде** ответ `400 invalid_code` содержит `attempts_remaining`; после исчерпания
  попыток сессия переходит в `failed`.
