---
title: pass_call — a robot dictates the code
sidebar_label: pass_call
sidebar_position: 5
---

# `pass_call` — a robot dictates the code

A robot calls the user's number and reads out the code; the user enters it on your side.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App as Your backend
    participant VH as Verificahub
    App->>VH: POST /v1/verify {method: pass_call}
    VH-->>App: request_id, code_length, status: sent
    VH-->>U: incoming call, robot dictates the code
    App-->>U: code input (code_length digits)
    U-->>App: entered code
    App->>VH: POST /v1/verify/check {request_id, code}
    VH-->>App: status: verified (or 400 invalid_code + attempts_remaining)
```

## What to watch for

- **The call has to be answered** — unlike [`flash_call`](./flash_call.md), where it drops by itself.
  Say so in your interface.
- **Billing is on initiation**, not on the conversation: the call is placed when you ask for it,
  whether or not the user picks up.
- **Useful as a fallback** where SMS does not arrive and the user has no messengers.
