---
title: telegram_otp — a code from Telegram
sidebar_label: telegram_otp
sidebar_position: 6
---

# `telegram_otp` — a code from Telegram

We generate a code and deliver it to Telegram through the official bot; the user enters it on your
side and you submit it for checking.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App as Your backend
    participant VH as Verificahub
    participant TG as Telegram
    App->>VH: POST /v1/verify {method: telegram_otp}
    VH-->>App: request_id, code_length, status: sent
    VH->>TG: code delivery
    TG-->>U: code in Telegram
    App-->>U: code input (code_length digits)
    U-->>App: entered code
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (or 400 invalid_code + attempts_remaining)
```

## What to watch for

- **The user needs Telegram on that number.** With no account there is nowhere to deliver the code —
  have a fallback method ready.
- **On a wrong code** the `400 invalid_code` response carries `attempts_remaining`; once attempts run
  out the session moves to `failed`.
