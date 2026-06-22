---
title: Overview and onboarding
sidebar_label: Overview
sidebar_position: 1
---

# Getting started

1. Request access on [verificahub.ru](https://verificahub.ru) — after review we issue API access and a test balance.
2. Get your API key in the dashboard.
3. Send your first number verification request.

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{ "phone_number": "+79991234567", "method": "reverse_flash_call" }'
```

Next, see the ready-made [use-case guides](../guides/reverse_flash_call) and answers in the [FAQ](../faq).
