---
title: max_otp — a code in the MAX messenger
sidebar_label: max_otp
sidebar_position: 7
---

# `max_otp` — a code in the MAX messenger

The code arrives as a message in MAX. No links, no extra steps: it is already delivered by the time
you get the response to `POST /v1/verify`.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App as Your backend
    participant VH as Verificahub
    participant MAX as MAX
    App->>VH: POST /v1/verify {method: max_otp}
    VH->>MAX: code delivery
    MAX-->>U: code in MAX
    VH-->>App: request_id, code_length, status: sent
    App-->>U: code input (code_length digits)
    U-->>App: entered code
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (or 400 invalid_code + attempts_remaining)
```

## What to watch for

- **Not to be confused with [`max_bot`](./max_bot.md).** There the user has to open the bot and share
  their number; here they do nothing — the message simply arrives. Most integrations want `max_otp`.
- **Show the code input straight away** after the `POST /v1/verify` response: the code is already sent.
- **The user needs MAX on that number** — otherwise there is nowhere to deliver it.
