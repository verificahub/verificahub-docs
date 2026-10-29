---
title: pass_call — робот диктует код голосом
sidebar_label: pass_call
sidebar_position: 5
---

# `pass_call` — робот диктует код голосом

Робот звонит на номер пользователя и проговаривает код; пользователь вводит его у вас.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: pass_call}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: входящий звонок, робот диктует код
    App-->>U: поле ввода кода (code_length цифр)
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (или 400 invalid_code + attempts_remaining)
```

## Что учесть

- **На звонок нужно ответить** — в отличие от [`flash_call`](./flash_call.md), где звонок
  сбрасывается. Предупредите пользователя об этом в интерфейсе.
- **Списание происходит при инициации**, а не по факту разговора: звонок размещается в момент
  запроса независимо от того, возьмёт ли пользователь трубку.
- **Подходит как запасной канал** там, где SMS не доходят, а мессенджеров у пользователя нет.
