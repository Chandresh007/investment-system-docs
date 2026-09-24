# Synthetic estimate qualification

- **Purpose:** deterministic Phase 2.5 estimate replay and regression tests.
- **Profile/version:** `synthetic_estimates` / `synthetic_estimates_v1`;
  qualification `synthetic-estimates-2026-09-22-v1`.
- **Tested endpoints:** local synthetic event payloads through raw archive,
  normalization, contributor replay, consensus, and feature publication.
- **PIT semantics:** fully controlled synthetic publication and correction times.
- **Revision behavior:** initiation, revision, withdrawal, correction, branching
  rejection, reporting evidence, and absolute fiscal target behavior are tested.
- **Coverage:** fixture scenarios only; no issuer/provider breadth claim.
- **Delisted support:** none and not relevant to the estimate adapter alone.
- **Licensing/rights:** fixtures are repository-owned synthetic data.
- **Historical replay eligibility:** only with `allow_synthetic=True`; resulting
  quality is always `DEVELOPMENT`.
- **Production status:** `SYNTHETIC_TEST_ONLY`.

No commercial analyst provider, contributor identity, historical coverage,
adjusted basis, correction semantics, publication completeness, or data rights
have been qualified. Today's consensus cannot reconstruct prior revisions.
