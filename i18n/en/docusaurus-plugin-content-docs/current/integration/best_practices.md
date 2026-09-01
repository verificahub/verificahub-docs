---
title: Best practices
sidebar_label: Best practices
sidebar_position: 4
---

# Best practices

A short checklist for a resilient production integration.

## Webhooks vs polling

- **Webhooks** are the primary way to learn the result: fast and without extra requests. Always
  [verify the signature](./webhooks.md) and make processing idempotent on `(request_id, event)`.
- **Polling** `GET /v1/verify/{request_id}` is a safety net (if a webhook didn't arrive) and the source of
  truth on mismatch. A sane interval is every 2–3 seconds until a terminal status, no faster.

## Don't show a code input blindly

A code isn't always needed. `reverse_flash_call` never has one; for `mts_id` it only appears on the
SMS-OTP fallback. Key off the signal, not the method:

- listen for the `verification.code_required` webhook, **or**
- poll `awaiting_code` in `GET /v1/verify/{request_id}` — `true` means "a code is needed now".

A hardcoded "enter the code" screen breaks `mts_id`. Details in [Verification flows](./flows.md).

## Remember `mts_id` is asynchronous

For `mts_id`, the `POST /v1/verify/check` response is `status: sent`, not `verified`: the code is relayed
to the operator and the outcome arrives a moment later by webhook or via polling. Don't treat `check` as
the final step.

## Don't hardcode tariffs

Prices change. Read them at runtime via `GET /v1/prices` (for integrators) or `GET /dashboard/prices`
(for the dashboard) and show the current values. The response contains only the client price, no internal
cost.

## Branch on `error_code`

Errors are RFC-7807 with a stable `error_code` field. Branch on it, not on the `detail` text:

| `error_code` | What to do |
| --- | --- |
| `insufficient_balance` (402) | Top up the balance; watch it via `GET /v1/balance` |
| `invalid_code` (400) | Show `attempts_remaining`, let the user retry |
| `not_pending` (409) | Session already resolved/expired, or the code was submitted too early |
| `not_found` (404) | Unknown `request_id` for your account |
| `rate_limit_exceeded` (429) | Wait for `Retry-After` and retry |
| `validation_error` (400) | Check field formats (E.164 number, 4–8 digit code) |

When initiation fails (`4xx`/`5xx` on `POST /v1/verify`) no charge is applied. For `max_bot`, billing is
charge-on-delivery: if the user never opens the bot or never shares their number, the code isn't delivered
and no charge is applied.

## Small things that save time

- **Numbers in E.164** (`+7…`). Normalize before sending.
- **Session TTL** — `expiry_seconds` (default 300, max 600). After it elapses the status is `expired`.
- **`request_id`** — your key for status, `check`, and webhook dedup; store it right away.
- **Secret stays server-side.** `api_secret` must not reach a browser or mobile app; rotate keys in the
  dashboard.
