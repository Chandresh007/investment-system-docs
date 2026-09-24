# Yahoo market qualification

- **Purpose:** development OHLCV, current exploratory research, connector tests.
- **Profile/version:** `yahoo_finance` / `yahoo_chart_v8`;
  qualification `yahoo-market-2026-09-22-v1`.
- **Tested endpoint:** `query1.finance.yahoo.com/v8/finance/chart/{ticker}` with
  explicit period bounds and interval; repository normalization is
  `yahoo_chart_v2`.
- **PIT semantics:** `HISTORICAL_AS_SEEN_TODAY`. Receipt gating prevents a download
  made today from appearing in an earlier operational snapshot, but it does not
  prove that returned historical values match what the provider showed then.
- **Revision behavior:** historical corrections and retrospective adjustment
  behavior are not versioned or qualified.
- **Coverage:** no measured complete interval, exchange, security-type, benchmark,
  failed-company, or delisted-security coverage.
- **Delisted support:** unverified. The profile has none of `DELISTED_HISTORY`,
  `SECURITY_LIFECYCLE`, `HISTORICAL_SECURITY_COVERAGE`, or `SURVIVORSHIP_SAFE`.
- **Licensing/rights:** storage, commercial use, derived-output, backup, and
  redistribution rights remain unverified.
- **Historical replay eligibility:** rejected. A development override is recorded
  in run provenance and forces quality `DEVELOPMENT`; it grants no PIT or
  survivorship claim.
- **Production status:** `DEVELOPMENT_ONLY`.

## Field contract

| Field | Canonical handling | Qualification |
| --- | --- | --- |
| `open`, `high`, `low`, `close` | Stored from `indicators.quote[0]` | Provider-reported basis, development only |
| `volume` | Stored from `quote.volume` | Share basis unverified, development only |
| adjusted close | Stored separately from `indicators.adjclose[0]` | Provider-derived, non-PIT-safe metadata; never a Phase 2.4 engine input |
| splits | Not ingested as a qualified canonical action feed | Unsupported for production |
| dividends | Not ingested as a qualified canonical action feed | Unsupported for production |
| ticker/merger/spinoff/delisting events | No qualified feed | Unsupported for production |

Phase 2.4 calculation correctness is separate from this evidence decision. The
frozen engine consumes `close`, reciprocal volume adjustments, and actions known
by the research timestamp. A regression deliberately gives `adjusted_close=1`
and `close=100` and proves the engine uses 100.

A replacement production action feed must provide announcement/publication time,
effective date, split ratio, cash distribution and share basis, symbol changes,
mergers, acquisitions, spinoffs, delistings and terminal proceeds, corrections,
and stable security identity. Prices need raw historical OHLCV, revision history,
session semantics, failed/delisted coverage, and verified rights. Yahoo currently
meets none of those production claims.
