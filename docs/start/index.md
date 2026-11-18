---
title: Обзор и подключение
sidebar_label: Обзор
sidebar_position: 1
---

# Быстрый старт

1. Зарегистрируйтесь в [личном кабинете](https://app.verificahub.ru/register) и подтвердите email.
2. Дождитесь активации аккаунта после проверки. Затем откройте «Интеграция → API-ключи»
   и получите пару `api_key` / `api_secret`.
3. Проверьте баланс через `GET /v1/balance` и доступные методы через `GET /v1/prices`.
   При необходимости пополните баланс и отправьте первый запрос.

Регистрация сама по себе не обещает тестовый баланс. MAX подключается отдельно в
[разделе управления аккаунтами MAX](../guides/max_accounts.md).

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -u api_key:api_secret \
  -H "Content-Type: application/json" \
  -d '{ "phone_number": "+79991234567", "method": "reverse_flash_call" }'
```

Дальше — готовые сценарии в разделе [Сценарии использования](../guides/reverse_flash_call) и ответы в [FAQ](../faq).
