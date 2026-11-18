---
title: Connect and manage MAX accounts
sidebar_label: MAX accounts
sidebar_position: 6
---

# MAX accounts

Use `max_otp` to send codes through your own MAX accounts. Your API credentials,
recipient phone number and `POST /v1/verify` stay the same. `max_bot` is a separate
bot/contact-sharing method and is unaffected by account setup.

## First connection

1. Open Integration → MAX in the [dashboard](https://app.verificahub.ru/integration?tab=max).
   Your Verificahub account must be active.
2. Choose Connect MAX or turn on the switch. Setup opens on a separate page.
   Confirm channel enablement.
3. Enter your MAX account's Russian mobile number and an optional name.
4. Scan the QR code in the MAX app. If you use the dashboard on your phone,
   display the code on another screen. Enter the two-factor password if requested.
5. Wait for connection success. You can then add more accounts.

An in-progress attempt can be resumed in the same browser tab after refresh or
returning to the account list. Leaving the page does not cancel the attempt;
use the separate cancel action. An expired attempt needs a new connection;
a refreshing QR should be awaited within the same attempt.

## Enabled versus ready to send

The switch permits MAX sending. Ready to send additionally requires at least
one currently usable connected account.

- An empty pool, expired sessions, disabled senders or cooldown of all senders
  stops sending while leaving the switch on.
- A usable account becoming available restores sending automatically.
- This mode uses only your accounts, without a shared fallback.
- Turning the switch off pauses new MAX codes while preserving connections.
  Turning it on resumes sending when a usable sender exists.

`GET /v1/prices` reports the account rate and method permission. It does not reserve
capacity or guarantee that a connected MAX sender is currently usable. Existing
VerificaHub ID method policies are not changed automatically.

## Manage accounts

The list shows phone, name and current status. Disable a sender, reconnect it,
forget a remembered password, or disconnect it. A previously disabled sender
requires authorization again after re-enabling.

Disconnect removes stored connection credentials from the service. It does not
promise to revoke the device session inside MAX; use the MAX app's device
settings when needed. Removing the last usable sender stops sending.

Account restrictions and blocking by MAX remain possible. Use accounts you are
authorized to manage and observe platform rules. Profile customization and its
available settings are controlled by the MAX app itself.

## API and billing

After setup call `POST /v1/verify` with `method: max_otp`. The user enters the code,
and your server checks it through `POST /v1/verify/check`.
See the [MAX verification flow](../integration/flows/max_otp.md).

Confirmed submission is billed at the account rate; acceptance does not prove
handset receipt or reading. If submission has an unknown outcome, do not
create a new verification automatically. Retain `request_id` from the error when
present and inspect its status. See [error handling](./errors_and_statuses.md).
