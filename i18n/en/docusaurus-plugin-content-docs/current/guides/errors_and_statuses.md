---
title: Statuses and error handling
sidebar_label: Statuses and errors
sidebar_position: 5
---

# Statuses and error handling

## Verification statuses

Every session has a status. It changes as the verification proceeds:

| Status | What it means | What to do |
| --- | --- | --- |
| `sent` | Verification initiated, waiting for the user | Poll the status |
| `delivered` | Code/call delivered to the user | Poll the status |
| `verified` | Number confirmed | Let the user through |
| `expired` | Session TTL elapsed (`expires_at`) | Offer to start over |
| `failed` | Verification could not complete | Offer another method or retry |

`verified`, `expired` and `failed` are terminal — no need to poll the session after them.

## HTTP response codes

| Code | When | Reaction |
| --- | --- | --- |
| `201` | Verification created successfully | Show the number / ask for the code |
| `200` | Successful status, balance or code-check request | — |
| `400` | Bad request: wrong code, bad number format, expired session | Show the user a clear error |
| `401` | Wrong `api_key` / `api_secret` | Check the credentials |
| `402` | Insufficient balance | Top up the balance |
| `404` | No session with that `request_id` | Check the identifier |

The error body uses the [Problem Details](https://datatracker.ietf.org/doc/html/rfc7807) format:

```json
{
  "type": "https://verificahub.ru/errors/insufficient-funds",
  "title": "Insufficient funds",
  "status": 402,
  "detail": "Not enough balance to initiate the verification."
}
```

## Recommendations

- **Timeouts.** If a session reaches `expired`, don't try to "push it through" — create a new one.
- **Idempotency on your side.** Store the `request_id` and don't create a new verification while the current one is active.
- **Clear messages.** On `400` during code entry, show "Invalid code" and let the user request a new one.
- **Balance monitoring.** Catch `402` and warn about a low balance in advance — see [Balance and usage](./balance_and_usage.md).
