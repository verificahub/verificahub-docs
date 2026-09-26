---
title: Dashboard changelog
sidebar_label: Dashboard changelog
sidebar_position: 41
---

# Dashboard changelog

What's changing in the customer dashboard — [app.verificahub.ru](https://app.verificahub.ru).
Public API changes have their [own changelog](./changelog.md).
Newest entries first.

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
