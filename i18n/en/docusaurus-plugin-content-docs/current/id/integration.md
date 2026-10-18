---
title: Integrating Verificahub ID
sidebar_label: Integration
sidebar_position: 2
---

# Integration

Four steps on your side: register a client, send the user to `/authorize`, receive them back and
exchange the code for the number. A working example:
[verificahub-id-integrator-sample](https://github.com/verificahub/verificahub-id-integrator-sample).

## 1. Client

In the Verificahub dashboard, create a sign-in client. You get a `client_id` and a `client_secret`.

- **`redirect_uri`** — the addresses the user returns to after signing in. They are compared
  **byte for byte**: no prefixes, no wildcards. On a mismatch Verificahub ID does not redirect the user
  and returns an error instead — this protects the code from interception.
- **Sign-in methods** — which methods to offer and in what order.
- **Branding** — what the user sees on the page:

| Field | What it looks like |
|---|---|
| `logoUrl` | Logo at the top of the page. If the file fails to load, the first letter of the name is shown instead. A transparent SVG or PNG works best. |
| `brandColor` | Color of the primary button and highlights. The button text color is picked automatically for contrast; if the color blends into the background in one of the themes, the default color is kept there. |
| `displayName` | Service name shown on the page. Defaults to the client name. |
| `defaultLocale` | `ru` or `en`. |
| `tosUrl`, `privacyUrl` | Links to your terms and privacy policy. |

Keep `client_secret` on the server only: it is needed to exchange the code and must never end up in a
browser or a mobile app.

## 2. Send the user to /authorize

Generate random `state` and `nonce` values on the server, store them in the user's session and
redirect the browser:

```http
GET https://id-api.verificahub.ru/authorize
    ?client_id=YOUR_CLIENT_ID
    &redirect_uri=https%3A%2F%2Fyour.app%2Fauth%2Fcallback
    &state=RANDOM_STATE
    &nonce=RANDOM_NONCE
```

| Parameter | Required | Purpose |
|---|---|---|
| `client_id` | yes | Your client. |
| `redirect_uri` | yes | One of the registered addresses, exact match. |
| `state` | no, but you need it | Returned unchanged — this is your CSRF protection. |
| `nonce` | no, but you need it | Included in the assertion — binds it to this sign-in. |

`/authorize` redirects the user to the sign-in page. A sign-in request lives for 10 minutes (by default).

## 3. Receive the user back

After sign-in, the browser returns to your `redirect_uri`:

```
https://your.app/auth/callback?code=…&state=RANDOM_STATE
```

Compare `state` with the stored value. If it doesn't match, abort the sign-in. `code` is single-use
and valid for 60 seconds (by default): exchange it right away.

## 4. Exchange the code for the number

Server-side only. The body is JSON in **camelCase** (`client_id` in snake_case is not recognized here):

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

Errors come back as `{ "error": "…", "message": "…" }`:

| HTTP | `error` | Cause |
|---|---|---|
| 400 | `invalid_request` | `clientId`, `clientSecret` or `code` is missing |
| 401 | `invalid_client` | Wrong client secret |
| 400 | `invalid_grant` | The code is wrong, expired, already used or issued to another client |
| 503 | `id_unavailable` | The service temporarily can't sign the assertion — retry later |

## 5. Verify the assertion

`assertion` is a JWT signed with RS256. Verify the signature against the keys at
`https://id-api.verificahub.ru/.well-known/jwks.json` (by the `kid` from the header), then check:

- `iss` = `https://id-api.verificahub.ru`;
- `aud` = your `client_id`;
- `exp` is in the future (an assertion lives for 5 minutes by default);
- `nonce` matches the one you sent in step 2;
- `phone_number_verified` = `true`.

| Claim | Value |
|---|---|
| `phone_number` | number in E.164 |
| `acr` | `high` — confirmed by the carrier (SIM push), `medium` — the user entered a code |
| `amr` | single-element array — the sign-in method |
| `nonce` | your `nonce` or `""` |

There is no `sub` claim: the user identifier is the phone number itself.

Node.js example with the [`jose`](https://github.com/panva/jose) library:

```js
import { createRemoteJWKSet, jwtVerify } from 'jose'

const JWKS = createRemoteJWKSet(new URL('https://id-api.verificahub.ru/.well-known/jwks.json'))

const { payload } = await jwtVerify(assertion, JWKS, {
  issuer: 'https://id-api.verificahub.ru',
  audience: process.env.VH_CLIENT_ID,
})
if (payload.nonce !== session.nonce || payload.phone_number_verified !== true) throw new Error('bad assertion')
// payload.phone_number is the user's verified number
```

## Checklist

1. `redirect_uri` is registered exactly as you pass it.
2. `state` is generated on the server and checked on return.
3. The code is exchanged for the number only on the server, with `client_secret`.
4. Trust the `assertion` signature, not the response body.
