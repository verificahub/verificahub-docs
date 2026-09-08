---
title: FAQ
sidebar_label: FAQ
sidebar_position: 30
---

# Frequently asked questions

## General

### What does Verificahub do?
It confirms that a phone number belongs to the user. You initiate a verification via the API, the user performs an action (a call or entering a code), and you get a verified number.

### Which verification methods are available?
Active: `reverse_flash_call`, `flash_call`, `telegram_otp`, `sms`. The `voice` method (voice code) is coming soon. The `max_bot` (a code in the MAX messenger) and `mts_id` (operator confirmation via MTS «Mobile ID») methods are available if enabled for your account. Full comparison and pricing — see [Methods and pricing](./guides/methods.md).

By how verification happens, methods split into two types:

- **Automatic** — `reverse_flash_call`: the user calls the number you show them, confirmation with no code entry.
- **Code entry** — `telegram_otp`, `sms`, `flash_call` (and `max_bot`, if enabled): the user enters a code and you confirm it via `POST /v1/verify/check`. For `flash_call` the code is the last digits of the incoming number; for `max_bot` the code arrives in MAX after the user shares their number in the bot.

### How do I get started?
Request access on [verificahub.ru](https://verificahub.ru), then get your API key in the dashboard — see [Getting started](./start/index.md).

## API requests

### What is the API base URL?
`https://api.verificahub.ru/v1`.

### How do I authenticate?
HTTP Basic: the `api_key:api_secret` pair. In curl — the `-u api_key:api_secret` flag. Send them only from your server, never from client-side code.

### What format should the number be in?
E.164, e.g. `+79991234567`. Numbers in responses are also normalized to E.164.

### What is `request_id`?
The verification session id. It's returned when you initiate and is used to check status (`GET /v1/verify/{request_id}`) or check a code.

### How long does a verification session live?
The TTL is set by the `expiry_seconds` parameter when initiating (a default applies if omitted). The exact end time comes in the `expires_at` field. After it, the status becomes `expired`.

### How do I know the number is verified?
For `reverse_flash_call`, poll `GET /v1/verify/{request_id}` until `status` is `verified`. For code-entry methods (`telegram_otp`, `sms`, `flash_call`, `max_bot`) confirmation is returned right away in the response to `POST /v1/verify/check`.

### Do I need to call `/v1/verify/check` for reverse flash-call?
No. With `reverse_flash_call` confirmation happens automatically from the incoming call. `/v1/verify/check` is only for code-entry methods (`telegram_otp`, `sms`, `flash_call`, `max_bot`).

## Pricing and billing

### How much does it cost?
Pay-as-you-go, at the method's rate. The price is fixed at initiation (the `cost` field); when you're actually charged depends on the method (see "When am I charged?"):

| Method | Price |
| --- | --- |
| `reverse_flash_call` | ₽0.25 |
| `flash_call` | ₽0.80 |
| `telegram_otp` | ₽0.90 |
| `max_bot` | per-account rate |
| `sms` | ₽3.00 |
| `mts_id` | ₽2.00 |
| `voice` | coming soon |

Prices are indicative; see current rates at [verificahub.ru](https://verificahub.ru).

### When am I charged?
The price is fixed at initiation and returned in the `cost` field, but **when you're actually charged depends on the method**:

- `reverse_flash_call` — only on a successful `verified`; the `expired` and `failed` statuses are **not charged**;
- `sms`, `flash_call`, `max_bot` — on delivery (the code/call actually went out); undelivered verifications are not charged;
- `telegram_otp`, `mts_id` — per attempt, at initiation.

So for `max_bot`, a user who never opens the bot or never shares their number is not charged.

### What happens at zero balance?
Initiation returns a `402` error. Track your balance via `GET /v1/balance` — see [Balance and usage](./guides/balance_and_usage.md).

### Where do I see usage and spending?
`GET /v1/usage` — a summary over a period: number of verifications, conversion, breakdown by method and status, and cost.

## Errors

### What do the `expired` and `failed` statuses mean?
`expired` — the user didn't confirm before the session ended. `failed` — verification couldn't complete. More in [Statuses and errors](./guides/errors_and_statuses.md).

### I'm getting `401` — what's wrong?
Wrong `api_key` or `api_secret`, or they weren't sent. Check the credentials and the auth scheme (Basic).

### I'm getting `400` when checking a code (Telegram / SMS / Flash Call).
The code is wrong or the session expired. Show the user a message and let them request a new code.
