---
title: sms — a code by SMS
sidebar_label: sms
sidebar_position: 3
---

# `sms` — a code by SMS

We generate a code and deliver it to the user by SMS; the user enters it on your side and you submit
it for checking.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App as Your backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: sms}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: SMS with the code
    App-->>U: code input (code_length digits)
    U-->>App: entered code
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (or 400 invalid_code + attempts_remaining)
```

## What to watch for

- **Code length comes in `code_length`.** Don't hard-code it into your markup; it is a platform setting.
- **On a wrong code** the `400 invalid_code` response carries `attempts_remaining`; once attempts run
  out the session moves to `failed`.
