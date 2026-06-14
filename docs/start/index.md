---
title: Обзор и подключение
sidebar_label: Обзор
sidebar_position: 1
---

# Быстрый старт

1. Оставьте заявку на [verificahub.ru](https://verificahub.ru) — после проверки
   мы выдадим доступ к API и тестовому балансу.
2. Получите API-ключ в личном кабинете.
3. Отправьте первый запрос на подтверждение номера.

```bash
curl -X POST https://api.verificahub.ru/v1/verify \
  -H "Authorization: Bearer sk_live_••••" \
  -d '{ "phone": "+79009009090", "method": "reverse_flash_call" }'
```

Подробнее о методах — в разделе [API](../api).
