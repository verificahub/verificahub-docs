---
title: max_bot — a code through a MAX bot
sidebar_label: max_bot
sidebar_position: 8
---

# `max_bot` — a code through a MAX bot

The difference from [`max_otp`](./max_otp.md) is that the user has to start the conversation. The
response to `POST /v1/verify` carries a `deep_link` (`https://max.ru/<bot>?start=<token>`); show it as
a link or a QR code. The user opens the bot, taps **"Share number"** — and only then does the code
arrive in MAX.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App as Your backend
    participant VH as Verificahub
    participant MAX as MAX
    App->>VH: POST /v1/verify {method: max_bot}
    VH-->>App: request_id, deep_link, code_length, status: sent
    App-->>U: link or QR for deep_link
    U->>MAX: opens the bot, taps "Share number"
    MAX->>VH: the user's contact
    VH->>MAX: code delivery → delivered
    MAX-->>U: code in MAX
    App-->>U: code input (code_length digits)
    U-->>App: entered code
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (or 400 invalid_code + attempts_remaining)
```

## What to watch for

- **The number must match.** The user has to share the **same** number the verification was started
  for, otherwise no code is sent.
- **Statuses tell the steps apart.** Before "Share number" the status is `sent`; after delivery it is
  `delivered` with `awaiting_code: true`.
- **Billing is charge-on-delivery.** If the user never opens the bot or never shares their number, no
  charge is applied.
- **The extra step costs conversion.** If you don't specifically need the bot dialogue, use
  [`max_otp`](./max_otp.md).
