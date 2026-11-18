---
title: Verification methods and pricing
sidebar_label: Methods and pricing
sidebar_position: 1
---

# Verification methods and pricing

Verificahub supports several ways to verify a number. They all use the same endpoint, `POST /v1/verify` — only the `method` field changes.

## Method comparison

| Method | `method` | How it verifies | Rate |
| --- | --- | --- | --- |
| Reverse flash-call | `reverse_flash_call` | The user calls the supplied number; verification by Caller ID | Per-account rate |
| Flash call | `flash_call` | The code is the last digits of the incoming number | Per-account rate |
| Telegram | `telegram_otp` | Code in a Telegram message | Per-account rate |
| MAX bot | `max_bot` | The bot sends a code after the user shares their number | Per-account rate |
| MAX code | `max_otp` | Code through connected MAX accounts | Per-account rate |
| SMS | `sms` | Code in an SMS message | Per-account rate |
| MTS ID | `mts_id` | SIM confirmation; SMS OTP on fallback | Per-account rate |
| Voice code | `pass_call` | A call dictating a code | Per-account rate |

Read current rates and account permissions from `GET /v1/prices`. Documentation
examples are not a live price list. `max_otp` requires [connected MAX accounts](./max_accounts.md).

## Two integration patterns

Methods split into two types by how verification happens.

### Automatic — no code entry

`reverse_flash_call` and the initial SIM check for `mts_id`. MTS SMS fallback requires a code. For `reverse_flash_call`: You show the user a number (`number_to_call`), they call it, and you poll the session status until `verified`.

→ [Reverse flash-call](./reverse_flash_call.md)

### Code entry

`telegram_otp`, `sms`, `flash_call`, `max_bot`, `max_otp`, `pass_call`, and MTS SMS fallback. The user receives or reads a code, enters it, and you confirm it via `POST /v1/verify/check`.

→ [Code verification](./code_verification.md)

:::tip Which method to choose
Reverse flash-call is the cheapest and needs no code entry. SMS is the most expensive but works on any phone without messengers — a good fallback channel.
:::

## Account availability and billing

`GET /v1/prices` includes `available` for each listed method; `false` means the
account is not permitted to use it. A request for a restricted method returns
403 `method_not_available` without a charge. For private MAX senders, permission
and current sender availability are separate; see [MAX accounts](./max_accounts.md).

`pass_call` is billed when the call request is accepted, even if the recipient
does not answer. Reverse flash-call is billed only on successful verification.
The quote in `cost` does not by itself prove that a charge was posted.
