---
title: telegram_otp — код из Telegram
sidebar_label: telegram_otp
sidebar_position: 6
---

# `telegram_otp` — код из Telegram

Мы генерируем код и доставляем его в Telegram официальным ботом; пользователь вводит код у вас, вы
отправляете его на проверку.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    participant TG as Telegram
    App->>VH: POST /v1/verify {method: telegram_otp}
    VH-->>App: request_id, code_length, status: sent
    VH->>TG: доставка кода
    TG-->>U: код в Telegram
    App-->>U: поле ввода кода (code_length цифр)
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (или 400 invalid_code + attempts_remaining)
```

## Что учесть

- **У пользователя должен быть Telegram на этом номере.** Если аккаунта нет, доставить код некуда —
  предусмотрите запасной метод.
- **При неверном коде** ответ `400 invalid_code` содержит `attempts_remaining`; после исчерпания
  попыток сессия переходит в `failed`.
