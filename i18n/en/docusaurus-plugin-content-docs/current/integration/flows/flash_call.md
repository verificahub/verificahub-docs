---
title: flash_call — the code is in the incoming number
sidebar_label: flash_call
sidebar_position: 4
---

# `flash_call` — the code is in the incoming number

We place a dropped call to the user's number, and **the code is the trailing `code_length` digits of
the incoming number**. The user reads them off the screen and enters them on your side.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App as Your backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: flash_call}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: dropped call (the number carries the code)
    App-->>U: code input (code_length digits)
    U-->>App: trailing digits of the incoming number
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (or 400 invalid_code + attempts_remaining)
```

## What to watch for

- **Tell the user what to enter.** "The last 4 digits of the number that called you" — without that
  hint conversion is noticeably below SMS.
- **The call does not need answering**; it drops by itself.
- **Not to be confused with [`reverse_flash_call`](./reverse_flash_call.md)**, where the user places
  the call and no code is typed at all.
