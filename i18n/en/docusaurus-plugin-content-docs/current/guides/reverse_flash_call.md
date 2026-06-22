---
title: Verifying a number with reverse flash-call
sidebar_label: Reverse flash-call
sidebar_position: 2
---

# Verifying a number with reverse flash-call

Reverse flash-call (`reverse_flash_call`) is the cheapest way to verify a number. The user makes a short call to the number you show them, and Verificahub confirms it automatically by Caller ID. No code entry needed.

## What the user sees

1. In your app the user taps "Verify number".
2. You show them a number to call.
3. The user calls — they can hang up immediately; the call is free for them.
4. The number is verified automatically within a few seconds.

## Step 1. Initiate the verification

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{
    "phone_number": "+79991234567",
    "method": "reverse_flash_call",
    "expiry_seconds": 300
  }'
```

Response `201 Created`:

```json
{
  "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "phone_number": "+79991234567",
  "method": "reverse_flash_call",
  "status": "sent",
  "cost": { "amount": 0.25, "currency": "RUB" },
  "expires_at": "2026-06-17T12:35:00Z",
  "number_to_call": "+79090000000",
  "code_length": 4
}
```

- `number_to_call` — the number to show the user with a "Call this number" label.
- `request_id` — the session id you'll use to check status.
- `expires_at` — when the session stops being valid.

## Step 2. Show the number to the user

Display `number_to_call` and ask them to make the call. The call is short — the user can hang up as soon as it starts dialing.

## Step 3. Wait for the result

Poll the session status until it becomes `verified` (or `expired`):

```bash
curl https://api.verificahub.ru/v1/verify/3fa85f64-5717-4562-b3fc-2c963f66afa6 \
  -u api_key:api_secret
```

```json
{
  "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "phone_number": "+79991234567",
  "method": "reverse_flash_call",
  "status": "verified",
  "cost": { "amount": 0.25, "currency": "RUB" },
  "created_at": "2026-06-17T12:30:00Z",
  "verified_at": "2026-06-17T12:30:42Z",
  "expires_at": "2026-06-17T12:35:00Z"
}
```

Once `status` is `verified`, the number is confirmed and you can let the user through.

:::tip
Poll the status every 1–2 seconds. Verification usually arrives within 10–30 seconds of the call.
:::

See also: [statuses and errors](./errors_and_statuses.md).
