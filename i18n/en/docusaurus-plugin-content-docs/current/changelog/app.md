---
title: Dashboard changelog
sidebar_label: Dashboard
sidebar_position: 2
slug: /changelog-app
---

# Dashboard changelog

What's changing in the customer dashboard — [app.verificahub.ru](https://app.verificahub.ru).
Public API changes have their [own changelog](./api.md).
Newest entries first.

## 1.5.0 — 2026-09-29

### Added

- **An Analytics tab on verifications.** It used to be a placeholder. Over 7, 30 or 90 days you
  now see how many were attempted and verified, the success rate and what was billed; a daily
  chart with the verified share; and breakdowns by status and by method with their shares. Days
  with no verifications are shown as empty, so the series is not compressed and a drop in traffic
  is visible.
- **The low-balance threshold is now visible and configurable.** The warning email used to fire
  at a fixed 100 ₽ with no way to change it. Settings → "Cabinet information" now shows the
  amount we write to you at, and lets you change it: keep the default, set your own, or turn
  the warning off entirely.
- **A low-balance warning inside the dashboard.** When the balance drops below your threshold, a
  strip appears above the page with a link to top up — not just an email.
- **Support chat inside the dashboard.** You can now write to us without leaving the dashboard
  or switching to email. Your name and address are filled in from your profile — visible, and
  editable before you send. The chat is available only once you are signed in, and signing out
  closes the conversation, which matters on a shared computer.
- **Quick buttons on the home screen** — "Analytics" and "Top up balance". Analytics opens on the
  right tab and can be linked to directly.
- **The breakdown behind each акт.** Every акт in "Invoice payment" now has a button that
  downloads an .xlsx breakdown — summary, by method, by status, by day, and the per-charge
  listing. Every акт has one.

### Fixed

- **The documentation links on the home screen.** "API documentation", "Quick start" and
  "Method reference" all led to the documentation home page instead of their own sections. Each
  now opens the right page, in the side panel, without taking you out of the dashboard.
- **Dropdowns and hints were cut off inside tables.** A select inside a dialog, and the receipt
  hint, could fall outside the table or window and become invisible. They are now always shown
  in full.
- **Field labels in the stacked table cards on phones** used the wrong face — Cyrillic fell back
  to a substitute.

### Changed

- The settings section "Account" is now called **"Cabinet information"**.
- **The side panel is wider**, and takes the whole screen on a phone or in landscape.
- **Organisation requisites moved to their own page.** On a computer they could be edited right
  on "Invoice payment", while on a phone they had a separate page. It is now the same
  everywhere: a card with the requisites and an "Edit" button, or "Fill in requisites" when they
  are not set yet.
- **Акты are no longer hidden behind requisites.** The monthly package reports your own
  consumption and needs no requisites, so акты and their breakdowns are visible to everyone who
  receives them. Requisites are still required to issue an invoice.

## 1.2.0 — 2026-09-27

### Added

- **A note about the new dashboard version.** After an update, the home screen shows a strip with
  the version number; clicking it opens this page in a side panel. The note appears once — as soon
  as you open or dismiss it, it stays away until the next update.
- **The changelog is reachable from anywhere.** The panel also opens from the profile menu
  ("What's new") and from Settings, next to the version number.

## 1.1.1 — 2026-09-26

### Fixed

- **The Settings icon.** It rendered as a shapeless figure instead of a gear — both in the bottom
  navigation on phones and in the profile menu.

## 1.1.0 — 2026-09-26

### Added

- **Why a verification did not pass.** The verification list and the dashboard now show what
  actually happened under the status: "Declined by subscriber", "Not delivered", "Wrong code",
  "No response", "Number unreachable". Previously "Failed" and "Expired" looked the same, though
  they call for different responses — repeating the same channel after a refusal rarely helps,
  whereas an undelivered code is worth retrying.

## 1.0.0 — 2026-09-26

### Added

- **Versioning and this changelog.** Settings now has an "About" block showing the version
  number — worth quoting when you contact support — and a link to this page.
- **Live method prices.** The "Methods" block on the home screen shows your account's real
  prices instead of a fixed list.

### Fixed

- **The Balance page would not open.** When a new kind of balance movement appeared, the page
  could fail to load. Unfamiliar movement types no longer break it.
- **The dashboard on phones.** The recent-verifications table was cut off and the cost column
  did not fit on screen — each record now shows as its own card. The language picker in the
  profile menu ran off the edge of the screen. Tall dialogs — the invoice preview, for example —
  were clipped in landscape with no way to scroll.

### Changed

- **The developer chat link** now points to an invite-only chat.
