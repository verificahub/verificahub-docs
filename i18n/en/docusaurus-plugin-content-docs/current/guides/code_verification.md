---
title: Code verification (Telegram, SMS, Flash Call)
sidebar_label: Code verification
sidebar_position: 3
---

# Code verification

The `telegram`, `sms` and `flash_call` methods follow the same flow: the user receives or reads a short code, enters it in your app, and you confirm the code via the API. Only the delivery channel differs.

| Method | Where the user gets the code |
| --- | --- |
| `telegram` | The code arrives as a Telegram message |
| `sms` | The code arrives over SMS |
| `flash_call` | A drop-call to the user's number; the code is the **last digits of the incoming number** |

## Step 1. Initiate the verification

Set the `method` you need (`sms`, `telegram` or `flash_call`):

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{
    "phone_number": "+79991234567",
    "method": "sms"
  }'
```

Response `201 Created`:

```json
{
  "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "phone_number": "+79991234567",
  "method": "sms",
  "status": "sent",
  "cost": { "amount": 3.0, "currency": "RUB" },
  "expires_at": "2026-06-17T12:35:00Z",
  "code_length": 4
}
```

`code_length` — how many digits the code has. Use it to configure the input field.

## Step 2. The user enters the code

- **Telegram / SMS** — the user receives the code in a message and types it in.
- **Flash Call** — the user gets a short drop-call; the code is the last `code_length` digits of the calling number. The user types them in (no need to answer).

## Step 3. Check the code

Submit the entered code together with the `request_id`:

```bash
curl -X POST https://api.verificahub.ru/v1/verify/check \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{
    "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "code": "1234"
  }'
```

Response `200 OK`:

```json
{
  "request_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "status": "verified",
  "phone_number": "+79991234567"
}
```

`status: "verified"` means the code is correct and the number is confirmed. If the code is wrong or the session expired, you get `400` — show the user a message and let them request a new code.

:::note
This flow does not apply to `reverse_flash_call` — there verification is automatic, with no code entry. See [Reverse flash-call](./reverse_flash_call.md).
:::
