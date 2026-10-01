---
title: API changelog
sidebar_label: API
sidebar_position: 1
slug: /changelog
---

# API changelog

Changes to the public API and to how verification methods behave.
Newest entries first.

## 2026-10-01

### Added

- **Credit limit: a permitted negative balance.** For accounts where this is agreed in a contract,
  verifications keep working once the balance goes below zero, down to the agreed floor. Every other
  account behaves exactly as before: verifications stop at a zero balance.

  `GET /v1/balance` gains two fields:

  - `credit_limit` — how far below zero the balance may go, or `null` if it may not;
  - `available` — what is actually left to spend: `balance` + `credit_limit`.

  **Watch `available`, not `balance`.** A request costing more than `available` is refused with
  `insufficient_balance` (402) — the error code has not changed. On an account with a credit limit
  `balance` can legitimately be negative, and that on its own is not an error.

  The low-balance email is now measured against remaining room rather than against zero, so a
  credited account is warned as it nears its floor instead of as its balance crosses zero.

## 2026-09-29

### Added

- **Detalization alongside the monthly act.** The monthly act is now accompanied by an `.xlsx`
  breakdown for the period — four sheets: summary, by verification method, by day, and a
  line-by-line list of billed verifications with date, masked number, method, status and amount.
  Download it in the dashboard; the link arrives with the act.

  Amounts come from what was actually charged, so the detalization total matches the act. Everything
  you were charged for is listed — **including verifications that did not succeed**: SMS is billed on
  send and Mobile ID per attempt, so billed rows with status "expired" or "failed" do appear, and that
  is expected. The summary calls those out on their own line, and a "by status" sheet breaks them down.

  Verifications you were never charged for are not listed, but the summary shows how many there were
  so the period reads in full.

## 2026-09-26

### Added

- **Why a verification failed.** `GET /v1/verify/{request_id}` and the dashboard verification list
  now return `failure_reason` on unsuccessful verifications: `no_response` (nobody acted before it
  expired), `declined` (the subscriber refused), `wrong_code` (attempts exhausted), `not_delivered`
  (the code or call never arrived), `unreachable` (the number cannot be served by this method), or
  `unknown`. The set is fixed and safe to branch on, and the list endpoint can filter by it
  (`?filter=failure_reason==not_delivered`).

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
