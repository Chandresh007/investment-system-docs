# Synthetic survivorship and classification qualification

- **Purpose:** deterministic foundation and Phase 2.6 lifecycle, taxonomy,
  classification, universe, identity, delisting, and peer-resolution tests.
- **Profile/version:** `synthetic_survivorship_fixture` /
  `synthetic_survivorship_v1`; qualification
  `synthetic-survivorship-2026-09-22-v2`.
- **Taxonomy:** fictional `SYNTHETIC_GICS_STYLE_V1` four-level structures only.
  The fixture contains no official GICS codes, names, definitions, assignments,
  or provider rights.
- **PIT semantics:** controlled effective intervals, knowledge timestamps,
  receipts, future-effective announcements, corrections, exits, re-entry,
  taxonomy releases, and sealed build membership.
- **Lifecycle coverage:** controlled primary-security, ticker/lifecycle, action,
  failed/delisted, and terminal-history scenarios used only by tests.
- **Capabilities:** synthetic historical replay/raw reproducibility, PIT actions,
  universe and security history, delisted/lifecycle and survivorship coverage,
  plus taxonomy-structure history and historical classification.
- **Coverage:** only the explicit fixture interval and fictional securities; no
  issuer, exchange, country, sector, or production breadth claim.
- **Licensing/rights:** repository-owned fictional fixtures. This grants no right
  to store or derive from proprietary taxonomy/provider data.
- **Historical replay eligibility:** only with `allow_synthetic=True`; every
  admitted result remains `DEVELOPMENT` even when the requested purpose is
  survivorship-safe testing.
- **Production status:** `SYNTHETIC_TEST_ONLY`.

The capability declarations prove that the runtime gate and algorithms can be
exercised end to end. They do not qualify a live security master, classification
vendor, market feed, official GICS history, or production/survivorship claim.
