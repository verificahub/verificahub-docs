---
title: Where to open the sign-in page
sidebar_label: Where to open the page
sidebar_position: 3
---

# Where to open the sign-in page

The flow is the same in any container: the code always arrives at your `redirect_uri`, and only your
server can exchange it. The browser — a tab, a popup or a WebView — is just where the user signs in.

| Container | Status | How |
|---|---|---|
| **Redirect in the same tab** | supported | The simplest option: redirect to `/authorize`, receive the user at `redirect_uri`. |
| **Popup window** | supported | Your page stays open. Recipe below. |
| **Mobile WebView** (WKWebView / Android WebView) | supported | Open `/authorize` in a WebView and intercept the navigation to `redirect_uri`. `embed=1` mode without the footer — see the parameters below. |
| **`<iframe>` on your site** | not allowed | The sign-in page can't be embedded in other sites — this protects against clickjacking. Use a popup instead: it shows the address bar, and that is what makes a sign-in page trustworthy. |

## Popup window

Open the window **directly in the click handler** (the browser blocks a window opened later) and point
it at `/authorize`, not at the sign-in page itself — it is `/authorize` that creates the sign-in request.

```js
const POPUP = 'width=460,height=720,menubar=no,toolbar=no,location=yes'

button.addEventListener('click', async () => {
  // your server builds the /authorize link: it has client_id, redirect_uri and the state it will check later
  const { url } = await fetch('/auth/start', { method: 'POST' }).then((r) => r.json())
  const popup = window.open(url, 'verificahub-id', POPUP)

  window.addEventListener('message', (e) => {
    if (e.origin !== 'https://id.verificahub.ru') return // always check the origin
    if (e.data?.source !== 'verificahub-id') return
    if (e.data.type === 'verified') showSpinner() // only a signal to the UI — see below
  })
})
```

The window then navigates to **your** `redirect_uri`, and your callback page closes it:

```html
<!-- your /auth/callback, opened in the popup after the code was exchanged on the server -->
<script>
  window.opener?.postMessage({ type: 'login-complete' }, window.location.origin)
  window.close()
</script>
```

Keep in mind that the user may close the window themselves: check `popup.closed` or simply keep the
sign-in button enabled.

### The `verified` message

Before redirecting back, the page posts to `window.opener` (and `window.parent`):

```js
{ source: 'verificahub-id', type: 'verified' }
```

It carries **neither the code nor the number**: the message is sent with `targetOrigin: '*'`, so any
window in the chain could read whatever it contains. It is a signal to the UI ("hide the button, show
a spinner"), not the result. The result is what your server received from `/exchange`.

## Page parameters

The page understands these parameters in its URL. **`/authorize` does not pass them through yet** —
support in `/authorize` is in development; until then the page picks the language from the client's
`defaultLocale` and follows the device theme.

| Parameter | What it does |
|---|---|
| `embed=1` | No footer — for a WebView where the app draws the frame. |
| `lang=ru\|en` | Page language. Defaults to the client's `defaultLocale`, otherwise Russian. |
| `theme=light\|dark` | Pins the theme — for apps whose theme differs from the system one. By default the page follows the device theme. |

## Sizes

- **Popup:** 460 × 720 is comfortable, it works from ~360 × 560. In a short window the page gets more
  compact, and if it still doesn't fit, it scrolls.
- **WebView:** give it at least 360 px of width. Input fields are at least 16 px, so iOS doesn't zoom
  the page on focus; screen notches and the home indicator are taken into account.
- `prefers-reduced-motion` and Windows high contrast mode are respected.

## What the page guarantees

- It never sees your `client_secret` and doesn't exchange the code for the number — only your server
  does that.
- No cookies: access is granted by a single-use link, so a lost link simply expires.
- The number entered on the page is kept only in that tab and only until sign-in completes — so the
  user can resume from code entry after a reload.
- Headers: HSTS, `Referrer-Policy: no-referrer`, `X-Content-Type-Options: nosniff`; embedding in other
  sites is blocked; the sign-in page is excluded from search indexing.
