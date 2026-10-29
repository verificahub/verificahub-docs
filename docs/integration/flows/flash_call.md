---
title: flash_call — код из номера входящего звонка
sidebar_label: flash_call
sidebar_position: 4
---

# `flash_call` — код из номера входящего звонка

Мы делаем звонок-сброс на номер пользователя, и **код — это последние `code_length` цифр входящего
номера**. Пользователь считывает их с экрана и вводит у вас.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: flash_call}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: звонок-сброс (номер содержит код)
    App-->>U: поле ввода кода (code_length цифр)
    U-->>App: последние цифры входящего номера
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (или 400 invalid_code + attempts_remaining)
```

## Что учесть

- **Объясните пользователю, что вводить.** «Последние 4 цифры номера, с которого поступил звонок» —
  без этой подсказки конверсия заметно ниже, чем у SMS.
- **Отвечать на звонок не нужно**, он сбрасывается сам.
- **Не путайте с [`reverse_flash_call`](./reverse_flash_call.md)**: там звонит пользователь и код не
  вводится вовсе.
