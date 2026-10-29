---
title: Как работают потоки подтверждения
sidebar_label: Обзор
sidebar_position: 1
---

# Как работают потоки подтверждения

Общая схема одна для всех методов: `POST /v1/verify` создаёт сессию, а результат вы узнаёте либо по
[вебхуку](../webhooks.md), либо опрашивая `GET /v1/verify/{request_id}`. Отличается только середина —
то, как пользователь проходит подтверждение.

Статусы сессии: `sent` → `delivered` (необязательный) → `verified`, либо `expired` (истёк TTL) /
`failed` (терминальная ошибка).

## Какой метод выбрать

Главный вопрос — нужно ли вам поле ввода кода.

| Без ввода кода | С вводом кода |
| --- | --- |
| [`reverse_flash_call`](./reverse_flash_call.md) — пользователь звонит сам | [`sms`](./sms.md) · [`flash_call`](./flash_call.md) · [`pass_call`](./pass_call.md) |
| [`mts_id`](./mts_id.md) — подтверждение на SIM, но с откатом на код | [`telegram_otp`](./telegram_otp.md) · [`max_otp`](./max_otp.md) · [`max_bot`](./max_bot.md) |

Какие методы доступны вашему аккаунту и сколько стоят — см.
[Методы подтверждения и тарифы](../../guides/methods.md).

## [`reverse_flash_call`](./reverse_flash_call.md) — без ввода кода

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
```

→ [Подробнее про `reverse_flash_call`](./reverse_flash_call.md)

## [`sms`](./sms.md) — код в SMS

Код приходит пользователю по SMS; он вводит его у вас, вы отправляете на проверку. Та же схема
«код → `/v1/verify/check` → `verified`» работает и у остальных методов с вводом кода.

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

→ [Подробнее про `sms`](./sms.md)

## [`flash_call`](./flash_call.md) — код из номера входящего звонка

Мы делаем звонок-сброс, и код — последние `code_length` цифр входящего номера. Пользователь
считывает их с экрана.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: flash_call}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: звонок-сброс (номер содержит код)
    App-->>U: поле ввода кода
    U-->>App: последние цифры входящего номера
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified
```

→ [Подробнее про `flash_call`](./flash_call.md)

## [`pass_call`](./pass_call.md) — робот диктует код голосом

Робот звонит и проговаривает код. В отличие от `flash_call`, на звонок нужно ответить.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь
    participant App as Ваш backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: pass_call}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: входящий звонок, робот диктует код
    App-->>U: поле ввода кода
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified
```

→ [Подробнее про `pass_call`](./pass_call.md)

## [`telegram_otp`](./telegram_otp.md) — код из Telegram

Код доставляет официальный бот Telegram.

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
    App-->>U: поле ввода кода
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified
```

→ [Подробнее про `telegram_otp`](./telegram_otp.md)

## [`max_otp`](./max_otp.md) — код в мессенджере MAX

Сообщение приходит само, без ссылок и действий со стороны пользователя.

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
    App-->>U: поле ввода кода
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified
```

→ [Подробнее про `max_otp`](./max_otp.md)

## [`max_bot`](./max_bot.md) — код через бота в MAX

Пользователь открывает бота по `deep_link` и жмёт «Поделиться номером» — код приходит только после
этого. Списание — по факту доставки.

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
    App-->>U: поле ввода кода
    U-->>App: введённый код
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified
```

→ [Подробнее про `max_bot`](./max_bot.md)

## [`mts_id`](./mts_id.md) — подтверждение через оператора

Два пути, и заранее не известно, какой сработает: обычно оператор подтверждает прямо на SIM, но при
неудаче происходит откат на SMS-OTP. **Не показывайте поле ввода кода сразу** — ждите вебхук
`verification.code_required` или флаг `awaiting_code: true`.

```mermaid
sequenceDiagram
    autonumber
    participant U as Пользователь (SIM)
    participant App as Ваш backend
    participant VH as Verificahub
    participant MTS as MTS «Мобильный ID»
    App->>VH: POST /v1/verify {method: mts_id}
    VH->>MTS: si-authorize (push на SIM)
    VH-->>App: request_id, status: sent
    alt Push подтверждён — код не нужен
        U->>MTS: подтверждение на SIM
        MTS-->>VH: результат (id_token)
        VH-->>App: webhook verification.verified
    else Откат на SMS-OTP — нужен код
        MTS-->>VH: уведомление об SMS-OTP
        VH-->>App: webhook verification.code_required
        App->>VH: POST /v1/verify/check {request_id, code}
        VH-->>App: status: sent (результат придёт асинхронно)
        VH-->>App: webhook verification.verified
    end
```

→ [Подробнее про `mts_id`](./mts_id.md)

## Узнать результат: вебхук или опрос

Оба способа рабочие, и их можно комбинировать (вебхук как основной, опрос — как страховка):

- **Вебхук** — мы сами присылаем событие на ваш URL (см. [Вебхуки](../webhooks.md)).
- **Опрос** — периодически запрашивайте `GET /v1/verify/{request_id}` до терминального статуса.

Источник истины при расхождениях — `GET /v1/verify/{request_id}`.
