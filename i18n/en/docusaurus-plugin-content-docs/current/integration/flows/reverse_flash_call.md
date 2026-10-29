---
title: reverse_flash_call — verification with no code typed
sidebar_label: reverse_flash_call
sidebar_position: 2
---

# `reverse_flash_call` — no code typed

The user calls a number we hand out, and we verify by caller id. Nobody types a code.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App as Your backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: reverse_flash_call}
    VH-->>App: request_id, number_to_call, status: sent
    App-->>U: "Call number_to_call"
    U->>VH: missed call to that number
    VH->>VH: caller-id match → verified
    VH-->>App: webhook verification.verified
    App->>VH: GET /v1/verify/{request_id} (if polling)
    VH-->>App: status: verified
```

Show the user `number_to_call` and wait for `verified` (by webhook or polling).

## What to watch for

- **`/v1/verify/check` isn't used for this method.** There is nothing to type.
- **The number is handed out per session.** Read it from the response every time; the next session may
  get a different one.
- **A withheld caller id never matches.** If the user calls with caller id hidden there is nothing to
  compare, and the session runs to `expired`.
