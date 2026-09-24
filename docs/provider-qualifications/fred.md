# FRED qualification

- **Purpose:** current macro context and connector development.
- **Profile/version:** `fred` / `latest_v1`; qualification
  `fred-2026-09-22-v1`.
- **Tested endpoint:** `fred/series/observations` without historical real-time
  bounds in the legacy connector.
- **PIT semantics:** `HISTORICAL_AS_SEEN_TODAY`. The API defaults to today's
  real-time period, so old observation dates do not establish old knowledge.
- **Revision behavior:** the legacy `EconomicObservation` identity stores one value
  per series/date and cannot represent all revisions.
- **Coverage:** configured series only; vintage depth, releases, missing values,
  timing, and third-party series rights are unqualified.
- **Delisted support:** not applicable.
- **Licensing/rights:** series-specific restrictions and retention rights remain
  unverified.
- **Historical replay eligibility:** rejected. Ordinary FRED cannot masquerade as
  vintage history.
- **Production status:** `DEVELOPMENT_ONLY`.

Use the canonical `EconomicVintage` path and the separate ALFRED qualification for
revision-aware work. Never infer vintages from successive latest-view downloads.
