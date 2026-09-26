---
title: Changelog
sidebar_label: Changelog
sidebar_position: 40
---

# Changelog

Changes to the public API and to how verification methods behave.
Newest entries first.

## 2026-09-27

### Added

- **Why a verification failed.** `GET /v1/verify/{request_id}` and the dashboard verification list
  now return `failure_reason` on unsuccessful verifications: `no_response` (nobody acted before it
  expired), `declined` (the subscriber refused), `wrong_code` (attempts exhausted), `not_delivered`
  (the code or call never arrived), `unreachable` (the number cannot be served by this method), or
  `unknown`. The set is fixed and safe to branch on, and the list endpoint can filter by it
  (`?filter=failure_reason==not_delivered`).

## 2026-09-26

### Added

- **Your own low-balance threshold.** `PUT /dashboard/account` now takes
  `low_balance_threshold` — the balance at which we email the account owner. It was previously a
  single platform-wide figure (100 ₽) that could not be changed. Send `null` to follow the
  platform default, or `0` to switch the warning off. The response returns both your value and the
  threshold actually in force (`effective_low_balance_threshold`).

### Fixed

- **Repeat low-balance warnings.** An account already below the threshold stopped receiving the
  email entirely, no matter how much further the balance fell. The warning now fires once on
  dropping below the threshold and re-arms after a top-up.

## 2026-09-24

### Changed

- **Only Russian mobile numbers are accepted.** `POST /v1/verify` now answers
  `400 region_not_supported` unless the number is a Russian mobile number (`+79…`). Previously any
  eleven digits starting with `7` were accepted, which let through Kazakh and Abkhazian numbers,
  Russian landlines, and typos such as `8890…` that normalisation turned into a plausible-looking
  `+7890…`. None of these can be delivered to, and none has ever verified — the request is now
  rejected outright, with no verification created and nothing charged.

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
