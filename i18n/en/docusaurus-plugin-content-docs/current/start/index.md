---
title: Overview and onboarding
sidebar_label: Overview
sidebar_position: 1
---

# Getting started

1. Register in the [dashboard](https://app.verificahub.ru/register) and verify your email.
2. Wait for account activation after review. Open Integration → API keys to obtain
   both `api_key` and `api_secret`.
3. Check `GET /v1/balance` and `GET /v1/prices`. Top up if needed, then send your
   first verification request.

Registration does not promise a test balance. MAX requires separate
[MAX account setup](../guides/max_accounts.md).

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{ "phone_number": "+79991234567", "method": "reverse_flash_call" }'
```

Next, see the ready-made [use-case guides](../guides/reverse_flash_call) and answers in the [FAQ](../faq).
