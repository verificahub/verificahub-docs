# Documentation audit — 18 November 2026

## Verified source and export

Current API source: `27c91f6`, including an OpenAPI-only correction to the initiation
operation description and its already-supported 403 response. Exported all three
Swagger documents through the real integration-test host with EF InMemory.
Production Swagger was not enabled and no production data was changed.

- Public document: 8 paths unchanged; schemas 33 → 38.
- Dashboard document: 45 → 56 paths; MAX lifecycle and pause routes were missing.
- The docs site consumes only the external document. Dashboard and internal
  snapshots stay in their respective API/app/admin repositories.
- Snapshot provenance is in `openapi/export-metadata.json`.

## Corrections prepared

- Onboarding now describes registration, email verification, review and the key/
  secret pair, without promising a test balance.
- Method and FAQ pages include pass_call and max_otp; no future voice placeholder.
- Indicative numeric price tables were replaced with per-account rate guidance.
- Added a RU/EN MAX account setup/management guide, keeping stable existing URLs.
- MAX submission acceptance is distinguished from receipt/reading; unknown
  submissions must not be retried as a new public verification automatically.
- Balance guidance includes available funds, credit limits and reservations;
  initiating a verification is no longer described as universally chargeable.
- Updated public examples and the Russian OpenAPI overlay against regenerated
  schemas. Updated stale dashboard/internal snapshots and frontend contract copies.
- Updated the docs README to the actual VPS hosting model.
- Updated the Russian software overview/functional pages to cover optional provider
  channels and private MAX credentials. Those pages have no existing EN counterparts.
- Refreshed the API repository's verbatim offer mirror from the canonical source.
  No canonical legal terms were changed.

## Release automation and remaining work

- Canonical legal pages still need a reviewed MAX amendment and a separate review
  of the consent basis/flow. English legal pages remain explicitly marked drafts.
- Backend release CI now exports the real Swagger document during tests and
  publishes the public artifact and provenance only after successful deployment.
  Local refresh uses `scripts/regen-openapi.sh` without a database or running API.
  Translations/examples remain curated overlays; new untranslated strings fall
  back to current English instead of showing stale Russian text.
- The site build currently allows translation checks to fail. Make that a release
  gate once the refreshed content is adopted.
- Real QR scanning/2FA completion requires a MAX account and is not a documentation
  build check. Do not claim that the interface or a disclaimer establishes MAX's
  permission for automated access.

## Validation

- Confirmed the live FAQ still described `voice` as forthcoming and the live MAX
  flow claimed delivery by the initiation response. The public downloaded spec
  contained the older 33-schema snapshot.
- Full Docusaurus RU and EN builds pass, including broken-link checks.
- Russian OpenAPI overlay: 158 translated strings; no missing/stale entries.
- All 85 public schema references resolve; only /v1 routes are published.
- Backend build: no warnings; 1,179 tests pass, 18 skipped. Swagger generation
  tests pass for external, dashboard and internal documents.
- App and admin lint/build checks pass; app retains its bundle-size advisory.
- Canonical legal documents remain unchanged; technical publication is handled
  separately from the legal review.
