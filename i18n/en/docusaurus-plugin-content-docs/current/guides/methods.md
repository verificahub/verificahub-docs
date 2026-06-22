---
title: Verification methods and pricing
sidebar_label: Methods and pricing
sidebar_position: 1
---

# Verification methods and pricing

Verificahub supports several ways to verify a number. They all use the same endpoint, `POST /v1/verify` — only the `method` field changes.

## Method comparison

| Method | `method` | How it verifies | Price per verification |
| --- | --- | --- | --- |
| Reverse flash-call | `reverse_flash_call` | The user calls our number; we confirm by Caller ID — no code | **₽0.25** |
| Flash Call | `flash_call` | A drop-call to the user's number; the code is the last digits of the incoming number | **₽0.80** |
| Telegram | `telegram` | The code arrives in Telegram via the official bot | **₽0.90** |
| SMS | `sms` | A classic code over SMS — works on any phone | **₽3.00** |
| Voice OTP | `voice` | A voice robot dictates the code | *coming soon* |

Prices are indicative; see current rates at [verificahub.ru](https://verificahub.ru).

## Two integration patterns

Methods split into two types by how verification happens.

### Automatic — no code entry

Only `reverse_flash_call`. You show the user a number (`number_to_call`), they call it, and you poll the session status until `verified`.

→ [Reverse flash-call](./reverse_flash_call.md)

### Code entry

`telegram`, `sms`, `flash_call` (and `voice` in the future). The user receives or reads a code, enters it, and you confirm it via `POST /v1/verify/check`.

→ [Code verification](./code_verification.md)

:::tip Which method to choose
Reverse flash-call is the cheapest and needs no code entry. SMS is the most expensive but works on any phone without messengers — a good fallback channel.
:::
