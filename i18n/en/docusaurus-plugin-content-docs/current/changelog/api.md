---
title: API changelog
sidebar_label: API
sidebar_position: 1
slug: /changelog
---

# API changelog

Changes to the public API and to how verification methods behave.
Newest entries first.

## 2026-10-07

### Added

- **`max_otp` — a new verification method: the code arrives as a MAX message. 2.00 ₽.** Nothing to
  open — no link, no bot, no app: the message simply arrives, like an SMS. The user types the code
  into `POST /v1/verify/check`, as with `sms`.

  ```json
  POST /v1/verify
  { "method": "max_otp", "phone_number": "+79991234567" }
  ```

  The message carries your website and name, so the recipient can tell whose code it is:

  ```
  2483 - ваш код подтверждения на ресурсе партнера https://example.ru (Пример (c) ООО «Пример»)
  ```

  The name comes from the account name, followed by the legal name from your requisites when they
  are filled in. Check in the dashboard that what is there is what you want shown to people.

  Charged on send: delivery is synchronous, so the response comes back after it. **A number that
  is not on MAX creates no verification and costs nothing** — `422`, as with any other
  undeliverable.

  **`max_bot` stays, and it is over six times cheaper — 0.30 ₽ against 2.00 ₽.** They are two
  different methods, not one replacing the other: with `max_bot` the user opens a bot from a link
  and shares their number, with `max_otp` they do nothing. Choosing between them trades price for
  that step.

- **Number lookups — two new services.** Separate from verification: a lookup does not prove
  anyone owns the number, it tells you about the number itself.

  - `POST /v1/lookup/number-info` — operator, region, and whether the number was ported. **0.80 ₽**
  - `POST /v1/lookup/activity` — how active the number is in the network, 0 to 1. **1.50 ₽**

  Both work on numbers from any RU operator. Every response carries `cost` — exactly what was
  charged; reconcile against it.

  **"No data for this number" is an answer, not an error:** it returns `200` with
  `status: "no_data"`, and we do not charge for it, nor for a number outside a service's operator
  coverage. Errors stay errors: malformed number (`400`), insufficient funds (`402`), service
  unavailable (`503`).

- **Check a list of numbers from the dashboard.** Upload a file of numbers (CSV, or simply one
  number per line) and get the results back as a file; up to 50,000 numbers at a time.

  You choose the check — number activity or number info. The price is the same as for a single
  check; the speed is not: an activity list is computed in one go and comes back quickly, while
  number info is collected a number at a time, so a large list takes hours rather than minutes.

  **You pay for the numbers that came back with an answer.** A number with no answer is not
  counted — the same rule as for a single check. The balance is checked before the job starts, so
  a job you cannot pay for is not half-run. A job that never completes costs nothing.

  Repeats in the file count once. Lines that are not a checkable number are left out of the job
  and returned to you with their line numbers — including the file's header row.

  The results stay downloadable for a limited time: the file holds your numbers, and we do not
  keep them longer than needed.

- **Lookup tariffs in `GET /v1/prices`.** A `lookups` list now sits alongside `prices`. They are
  two separate lists and should stay separate: a verification is charged at its own milestone
  (send, attempt or success, depending on the method), a lookup per answer. A service with no live
  tariff is not listed.

- **Lookup usage in `GET /v1/usage`.** A new `lookups` block:

  ```json
  "lookups": {
    "total": 40,
    "billable": 37,
    "total_cost": { "amount": 31.10, "currency": "RUB" },
    "by_method": { "number_info": 25, "activity_score": 15 },
    "by_status": { "ok": 37, "no_data": 3 }
  }
  ```

  The report's top-level fields are still verifications only — lookups are **not** included in
  them. Spend for the range is `total_cost` **+** `lookups.total_cost`. `billable` is how many
  requests were charged for; it is below `total` by exactly those with no data for the number.

- **Lookup services in the reference data** from `GET /meta` and `GET /config`: a separate
  `lookups` list, plus `lookup_statuses` and `bulk_lookup_statuses` in `/config`.

### Changed

- **Number lookups are included in the акт and its detalization.** They count towards the акт's
  amount alongside verifications, and the detalization gives them their own "Проверки номеров"
  sheet: a single lookup as a row with its masked number, a bulk job as one row naming the file
  and how many of its numbers were charged. The summary splits the total into verifications and
  number lookups.

  Amounts on акты for past periods are not recalculated.

- **`max_bot` is now labelled "MAX Bot" in the reference data, and "MAX" means `max_otp`.** Both
  methods used to be shown as "MAX", which made it impossible to pick the right one by name. The
  values (`max_otp`, `max_bot`) are unchanged — only the labels in `GET /meta` and `GET /config`
  moved.

### Fixed

- **Activity checks answered `503` instead of "no data".** When a number has no activity score,
  `POST /v1/lookup/activity` returned `503 lookup_unavailable` — "try again later" for a request
  whose answer will not change. It now returns what was promised: `200` with
  `status: "no_data"` and `cost` of 0.

  There is no point retrying: the operator has no activity score for that number. **A number-info
  check on the same number may well answer** — it is a property of the service, not of the number,
  so checking it with the other service is worth doing.

- **Bulk activity checks: the first number of the list was not checked, and numbers with no answer
  reached the invoice.** The job looked complete, but its result was missing the first row, and the
  cost was computed from how many numbers were processed rather than how many answered.

  **You now pay only for numbers that came back with an answer** — the same rule as a single
  check. A job shows both counts: `numbers_processed` for how many were worked through,
  `numbers_billed` for how many were charged.

  Jobs that ran before the fix are worth running again: their result file is missing its first row.
  If one of them was charged more than it has answers for, tell us and we will refund the
  difference.

## 2026-10-02

### Fixed

- **Invoice PDF download in the dashboard.** The download button on a счёт returned an
  error and no file came through. Fixed — the invoice now downloads, and arrives with a
  meaningful name ("Счет № 3 от 29.09.26.pdf" instead of "invoice-3.pdf").

  We also keep our own copy from the moment a счёт is issued, so it **stays downloadable
  for 30 days** regardless of how long the file link itself lasts. If you need an invoice
  for accounting beyond that, download and keep it within the window.

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
