---
title: Подключение Verificahub ID
sidebar_label: Подключение
sidebar_position: 2
---

# Подключение

Четыре шага на вашей стороне: зарегистрировать клиента, отправить пользователя на `/authorize`,
принять его обратно и обменять код на номер. Готовый пример —
[verificahub-id-integrator-sample](https://github.com/verificahub/verificahub-id-integrator-sample).

## 1. Клиент

В кабинете Verificahub создайте клиента для входа. Вы получите `client_id` и `client_secret`.

- **`redirect_uri`** — адреса, куда пользователь вернётся после входа. Сравниваются **побайтово**:
  без префиксов и масок. При несовпадении Verificahub ID не перенаправляет пользователя, а отвечает
  ошибкой — это защита от перехвата кода.
- **Способы входа** — какие способы и в каком порядке предлагать.
- **Оформление** — что увидит пользователь на странице:

| Поле | Как выглядит |
|---|---|
| `logoUrl` | Логотип наверху страницы. Если файл не загрузится, вместо него появится первая буква названия. Лучше прозрачный SVG или PNG. |
| `brandColor` | Цвет основной кнопки и выделения. Цвет текста на кнопке подбирается по контрасту автоматически; если цвет сольётся с фоном в одной из тем, там останется стандартный. |
| `displayName` | Название сервиса на странице. По умолчанию — имя клиента. |
| `defaultLocale` | `ru` или `en`. |
| `tosUrl`, `privacyUrl` | Ссылки на ваши условия и политику конфиденциальности. |

`client_secret` храните только на сервере: он нужен для обмена кода и не должен попадать в браузер
или мобильное приложение.

## 2. Отправьте пользователя на /authorize

Сгенерируйте на сервере случайные `state` и `nonce`, сохраните их в сессии пользователя и
перенаправьте браузер:

```http
GET https://id-api.verificahub.ru/authorize
    ?client_id=YOUR_CLIENT_ID
    &redirect_uri=https%3A%2F%2Fyour.app%2Fauth%2Fcallback
    &state=RANDOM_STATE
    &nonce=RANDOM_NONCE
```

| Параметр | Обязательный | Назначение |
|---|---|---|
| `client_id` | да | Ваш клиент. |
| `redirect_uri` | да | Один из зарегистрированных адресов, точное совпадение. |
| `state` | нет, но нужен | Вернётся без изменений — по нему вы защищаетесь от CSRF. |
| `nonce` | нет, но нужен | Попадёт в подтверждение — привязывает его к этому входу. |

`/authorize` перенаправит пользователя на страницу входа. Запрос на вход живёт 10 минут (по умолчанию).

## 3. Примите пользователя обратно

После входа браузер вернётся на ваш `redirect_uri`:

```
https://your.app/auth/callback?code=…&state=RANDOM_STATE
```

Сверьте `state` с сохранённым. Если не совпадает — прервите вход. `code` одноразовый и действует
60 секунд (по умолчанию): обменяйте его сразу.

## 4. Обменяйте код на номер

Только с сервера. Тело — JSON в **camelCase** (`client_id` в snake_case здесь не распознается):

```http
POST https://id-api.verificahub.ru/exchange
Content-Type: application/json

{ "clientId": "YOUR_CLIENT_ID", "clientSecret": "YOUR_CLIENT_SECRET", "code": "…" }
```

```json
{
  "assertion": "eyJhbGciOiJSUzI1NiIsImtpZCI6ImlkLXNpZy0xIiwidHlwIjoiSldUIn0…",
  "phoneNumber": "+79991234567",
  "phoneNumberVerified": true,
  "acr": "medium",
  "method": "sms"
}
```

Ошибки приходят как `{ "error": "…", "message": "…" }`:

| HTTP | `error` | Причина |
|---|---|---|
| 400 | `invalid_request` | Не передан `clientId`, `clientSecret` или `code` |
| 401 | `invalid_client` | Неверный секрет клиента |
| 400 | `invalid_grant` | Код неверный, истёк, уже использован или выдан другому клиенту |
| 503 | `id_unavailable` | Сервис временно не может подписать подтверждение — повторите позже |

## 5. Проверьте подтверждение

`assertion` — JWT, подписанный RS256. Проверьте подпись по ключам
`https://id-api.verificahub.ru/.well-known/jwks.json` (по `kid` из заголовка), затем:

- `iss` = `https://id-api.verificahub.ru`;
- `aud` = ваш `client_id`;
- `exp` в будущем (подтверждение живёт 5 минут по умолчанию);
- `nonce` совпадает с отправленным на шаге 2;
- `phone_number_verified` = `true`.

| Claim | Значение |
|---|---|
| `phone_number` | номер в E.164 |
| `acr` | `high` — подтверждено оператором (SIM-push), `medium` — пользователь ввёл код |
| `amr` | массив из одного элемента — способ входа |
| `nonce` | ваш `nonce` или `""` |

Поля `sub` нет: идентификатор пользователя — сам номер телефона.

Пример на Node.js с библиотекой [`jose`](https://github.com/panva/jose):

```js
import { createRemoteJWKSet, jwtVerify } from 'jose'

const JWKS = createRemoteJWKSet(new URL('https://id-api.verificahub.ru/.well-known/jwks.json'))

const { payload } = await jwtVerify(assertion, JWKS, {
  issuer: 'https://id-api.verificahub.ru',
  audience: process.env.VH_CLIENT_ID,
})
if (payload.nonce !== session.nonce || payload.phone_number_verified !== true) throw new Error('bad assertion')
// payload.phone_number — подтверждённый номер пользователя
```

## Чек-лист

1. `redirect_uri` зарегистрирован точно в том виде, в каком вы его передаёте.
2. `state` генерируется на сервере и сверяется при возврате.
3. Код меняется на номер только на сервере, с `client_secret`.
4. Доверяйте подписи `assertion`, а не телу ответа.
