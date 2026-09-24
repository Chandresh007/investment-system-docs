# ALFRED / FRED real-time qualification

- **Purpose:** macro vintage ingestion and replay.
- **Profile/version:** `alfred` / `alfred_realtime_v1`; qualification
  `alfred-2026-09-22-v1`.
- **Tested endpoint contract:** `fred/series/observations` with explicit
  `realtime_start`, `realtime_end`, `observation_start`, `observation_end`,
  `output_type=1`, offset, limit, and JSON. Raw pages and sanitized request
  provenance are archived. Pagination count/offset mismatch fails.
- **PIT semantics:** the official real-time interval is preserved. Because its
  revision date lacks an intraday release time, normalization uses the next UTC
  midnight as conservative `available_at`. Exact release timing remains a series-
  level qualification task.
- **Revision behavior:** each observation date plus `realtime_start` is a distinct
  append-only vintage. Exact decimal values and explicit missing releases persist.
- **Coverage:** not measured against live credentials or representative series.
  Pagination, vintage depth, missing releases, release lag, and corrections still
  need live qualification.
- **Delisted support:** not applicable.
- **Licensing/rights:** series-specific rights, retention, backups, and derived use
  remain unverified.
- **Historical replay eligibility:** rejected by the catalog until live evidence,
  coverage, timing, and rights are reviewed. Synthetic macro replay requires an
  explicit synthetic opt-in and always records `DEVELOPMENT` quality.
- **Production status:** `LIVE_ALFRED_CONNECTOR_PENDING`.

Mocked official-shape fixtures prove initial GDP 2.0 before the revision and 2.5
after it, without network-dependent tests. This validates the connector-to-schema
contract; it does not qualify the live service.

Official semantics: [real-time periods](https://fred.stlouisfed.org/docs/api/fred/realtime_period.html),
[series observations](https://fred.stlouisfed.org/docs/api/fred/series_observations.html),
and [vintage dates](https://fred.stlouisfed.org/docs/api/fred/series_vintagedates.html).
