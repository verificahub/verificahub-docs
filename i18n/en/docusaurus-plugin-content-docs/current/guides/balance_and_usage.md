---
title: Balance and usage
sidebar_label: Balance and usage
sidebar_position: 4
---

# Balance and usage

Verificahub is prepaid (pay-as-you-go): each initiated verification is charged at the method's rate. Two endpoints help you track spending.

## Current balance

```bash
curl https://api.verificahub.ru/v1/balance \
  -u api_key:api_secret
```

```json
{
  "balance": { "amount": 1500.0, "currency": "RUB" }
}
```

:::tip
Check the balance on a schedule and set up an alert when it drops below a threshold — with a zero balance you can't initiate a verification (error `402`).
:::

## Usage over a period

`GET /v1/usage` returns a summary over a date range: how many verifications were initiated, how many succeeded, the conversion rate and the total cost.

```bash
curl "https://api.verificahub.ru/v1/usage?from=2026-06-01&to=2026-06-17" \
  -u api_key:api_secret
```

```json
{
  "from": "2026-06-01",
  "to": "2026-06-17",
  "total_verifications": 1280,
  "successful": 1205,
  "success_rate": 0.9414,
  "total_cost": { "amount": 712.5, "currency": "RUB" },
  "by_method": { "reverse_flash_call": 900, "telegram": 200, "sms": 120, "flash_call": 60 },
  "by_status": { "verified": 1205, "expired": 60, "failed": 15 },
  "daily": [
    { "date": "2026-06-16", "total": 95, "verified": 90 },
    { "date": "2026-06-17", "total": 60, "verified": 57 }
  ]
}
```

What to do with it:

- `success_rate` — the share of verified attempts. A drop points to funnel issues.
- `by_status` — breakdown by reason. Lots of `expired` means users don't confirm in time (raise `expiry_seconds` or simplify the prompt); lots of `failed` is worth investigating case by case.
- `by_method` — which methods are in use.
- `daily` — for per-day charts (days with no activity are omitted).

The `from` and `to` parameters are optional: without them a default range is returned.
