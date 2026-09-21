---
title: Changelog
sidebar_label: Changelog
sidebar_position: 40
---

# Changelog

Changes to the public API and to how verification methods behave.
Newest entries first.

## 2026-09-21

### Fixed

- **`mts_id`, Tele2 and MegaFon subscribers.** SMS-code confirmation failed
  and the verification expired even when the code was entered correctly. The
  method now works for subscribers of every operator.
- **Resubmitting an already-accepted code no longer costs an attempt.** Once
  the operator has accepted a code, a repeated `POST /v1/verify/check` returns
  `409 not_pending` instead of `400 invalid_code`, and no attempt is spent.
- **The `verification.failed` event** is now delivered to your webhook for the
  `mts_id` method as well. Previously, running out of attempts on this method
  was only visible via `GET /v1/verify/{request_id}`.

### Note for integrators

- With `mts_id`, a code is only ever entered on the SMS-OTP leg, which is used
  when SIM-push confirmation is unavailable. There is nothing to enter during
  SIM-push, and `POST /v1/verify/check` returns `409 not_pending` there.
- On the SMS-OTP leg, a `200` means the operator **accepted** the code; the
  response body still reports `sent`. The outcome arrives as a
  `verification.verified` event (typically within a second), or via
  `GET /v1/verify/{request_id}`.

### Billing

- Verifications affected by technical issues on our side have been
  recalculated. Balances were credited and the affected integrators notified.
