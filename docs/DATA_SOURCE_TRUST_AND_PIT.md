# Data-source trust and point-in-time strategy

Review date: 2026-09-22. Source classifications describe permitted use, not vendor
marketing. An endpoint accepting historical dates is not proof of historical knowledge.
Source-contract findings below are separate from frozen mathematical definitions.

## Time vocabulary

| Time | Meaning | Example |
| --- | --- | --- |
| Event/effective date | When something economically happens | Split takes effect June 10 |
| Period date | Interval measured by a fact | Quarter ends March 31 |
| Publication date/time | Source distributes a specific version | Results released May 5 at 16:05 New York |
| `available_at` | Earliest conservatively justified use under a versioned policy | After release/dissemination, or first receipt if uncertified |
| `retrieved_at` | System actually received this payload | Download on May 6 |
| Provider revision | Later version/correction, with its own publication and receipt times | May 8 correction to the May 5 release |

Distinguish market-knowable historical reconstruction from actual operational
knowledge. The latter cannot precede receipt. Certified historical publication
may support the former, but only in an explicitly reconstructed, pinned build.
Date-only evidence needs a conservative documented timezone boundary; never assume
midnight at the beginning of the date. Preserve precision and uncertainty.

## Trust classes

- **AUTHORITATIVE PIT:** original authoritative evidence and verified timing for the precise claimed fact.
- **PIT IF USED CORRECTLY:** historical versions are potentially available, but selection/normalization determines validity.
- **CURRENT DATA ONLY:** suitable for present/as-retrieved context, not past knowledge.
- **HISTORICAL-AS-SEEN-TODAY:** old observation dates with today's revised representation.
- **ARCHIVE-OURSELVES:** preserve successive receipts and changes prospectively; cannot repair unobserved earlier history.
- **UNQUALIFIED:** timing, rights, semantics or coverage lack sufficient evidence.

| Source / use | Classification now | Qualification condition |
| --- | --- | --- |
| EDGAR original accession filing | AUTHORITATIVE PIT for filing content with verified dissemination; PIT IF USED CORRECTLY for extraction | Acceptance/availability/timezone validated; original bytes and accession retained |
| SEC Company Facts current response | HISTORICAL-AS-SEEN-TODAY aggregate; PIT IF USED CORRECTLY through accession evidence; ARCHIVE-OURSELVES | Original filing context, revisions and comparative facts verified before historical use |
| FRED current connector | HISTORICAL-AS-SEEN-TODAY | No vintage parameters currently requested |
| ALFRED / FRED real-time queries | Connector implemented; `LIVE_ALFRED_CONNECTOR_PENDING` | Vintage selection, release lag, live coverage and series rights established |
| Yahoo chart historical prices | HISTORICAL-AS-SEEN-TODAY; UNQUALIFIED as raw PIT input | Price/volume adjustment basis and action history proven per endpoint/version |
| yfinance | Same upstream limitations; not currently used by this connector | Pin library/options and qualify upstream semantics; wrapper adds no PIT guarantee |
| News RSS | CURRENT DATA ONLY / ARCHIVE-OURSELVES | Archived original and revised items, publication evidence, retrieval and retention rights |
| Synthetic analyst estimates | Qualified synthetic test contract only | Proves code behavior, not commercial coverage or rights |
| Today's commercial consensus | CURRENT DATA ONLY | Cannot reconstruct historical contributor revisions |
| Future analyst event archive | UNQUALIFIED until certified; then PIT IF USED CORRECTLY | Full qualification checklist below |
| Corporate actions | UNQUALIFIED production feed; internal schema/test fixtures exist | Announcement/knowledge/effective times, corrections and share basis |
| Current company/ticker/classification metadata | CURRENT DATA ONLY | Historical identifiers and bitemporal membership required |
| Future institutional/alternative sources | UNQUALIFIED | Individual contracts required; see risk register |

## SEC: good evidence, incomplete current qualification

SEC's APIs aggregate XBRL across filings and are updated as submissions are
disseminated; this is not a guarantee that today's aggregate reproduces every
past API response. Preserve accession-specific evidence and original bytes.
[SEC API documentation](https://www.sec.gov/search-filings/edgar-application-programming-interfaces)

Acceptance timestamps remain appropriate evidence for filing arrival, but acceptance
and public dissemination are not identical instants. Verify timezone and delay from
archived headers/provider evidence rather than assuming a date is an intraday timestamp.
[SEC webmaster FAQ](https://www.sec.gov/about/webmaster-frequently-asked-questions)

The former filing-midnight defect is closed. Filing and fact rows now carry an
explicit `available_at` and policy; the resolver uses exact aware acceptance
evidence when present and next New York midnight for date-only evidence. Company
Facts entries link to their accession filing when that evidence exists. The
aggregate Company Facts profile still cannot establish every historical API state,
so broad live replay remains blocked despite the corrected selection semantics.

Late filings become visible when released, not at period end. Amendments and
restatements are later knowledge, never replacements backdated to the original
report. Comparative facts must be linked to their actual accounting context:
copying `fy/fp` from an aggregate entry does not alone establish that every included
comparative period has that fiscal identity. Qualify with accession-level contexts,
non-calendar issuers and fiscal-year changes; do not infer quarters from month.
This is an unresolved semantic qualification risk, not a claimed new formula fix.

Earnings releases/8-K exhibits may precede 10-Q/10-K and need independently archived
reporting evidence. Phase 2.5's primary-reporting projection is conservative, but
no live reporting adapter establishes comprehensive coverage. Absence of a filing
is not proof that a quarter remains unreported.

## Yahoo and the Phase 2.4 boundary

The code calls Yahoo's chart endpoint directly, not yfinance. The normalizer reads
`indicators.quote[0].close`; the adjustment engine uses that `close` and never
`adjusted_close`. It filters price receipts by research cutoff, and corporate
actions by availability (falling back to creation time when null).

Yahoo describes adjusted close as incorporating splits and distributions.
The help search result was accessible; direct retrieval returned HTTP 429 during
this review. It does not certify the exact chart endpoint's raw-close semantics.
[Yahoo adjusted-close explanation](https://help.yahoo.com/kb/SLN28256.html)

**Conclusion: the architecture is conditionally protective, not end-to-end proven.**
Raw-as-traded prices + compatible volume + PIT-known actions → split-continuous
prices → TRI is a sound boundary only when the inputs truly meet it. Calling a
field `close` or setting `auto_adjust=False` in another library does not establish
that upstream history lacks retrospective split normalization. Double adjustment
is possible if normalized history is fed to the raw-price contract. Uniform scaling
may cancel in some ratios; that does not prove dollar volume or dividend math safe.

Current retrieval gating correctly excludes a 2026 download from a 2021 query;
this yields missing history, not a usable 2021 backtest. Never “fix” that by
backdating receipts. `MarketPrice` uniqueness omits provider/build/version and the
normalizer uses conflict-do-nothing: this is not a versioned price correction store.
The current connector/normalizer does not establish an ingested, qualified corporate-
action archive. Dividend amounts must share the normalized price's share basis,
including later splits. Unsupported actions require explicit exclusion coverage.

Qualification tests must independently reconcile archived prices/volumes before
and after splits, reverse splits, dividends, split-after-dividend combinations,
provider corrections and delisting proceeds. Compare repeated historical downloads
and an independent qualified reference; preserve hashes and library/endpoint/options.
Also qualify timestamps/session-close meaning and benchmark inception/history.

Yahoo remains useful for development and exploratory current context subject to
applicable terms. It is not certified institutional-quality PIT data. Augment or
replace it for historical claims with a licensed source proving raw prices,
revision history, action knowledge timestamps and delisted coverage. Keep the
canonical provider interface; do not select a vendor merely by brand.

yfinance's maintainers describe it as unaffiliated with Yahoo and direct users to
Yahoo's terms; its software license does not grant market-data redistribution rights.
Pin adjustment/repair options if it is introduced. [yfinance project](https://github.com/ranaroussi/yfinance)

## FRED and ALFRED

`FREDConnector.get_observations` sends only `series_id`, so it requests the API's
default current real-time view. Stored `realtime_start/end` fields do not cure
this: `EconomicObservation` is unique by series/date and normalization ignores
conflicting rows. It cannot preserve successive vintages as separate observations.
FRED documents the default as today's knowledge and supports historical real-time
intervals. [FRED real-time semantics](https://fred.stlouisfed.org/docs/api/fred/realtime_period.html)

The canonical `EconomicVintage` path and explicit ALFRED connector now use a
series/date/vintage identity, bounded real-time and observation ranges, archived
raw pages, and conservative date-level availability. Mocked official-shape tests
prove an initial value and a later revision without network access. Live series
coverage, release lag, pagination at scale, credentials, and rights are still
unverified, so `LIVE_ALFRED_CONNECTOR_PENDING` remains blocked from historical
replay. Vintage dates identify new/revised observations.
[FRED vintage endpoint](https://fred.stlouisfed.org/docs/api/fred/series_vintagedates.html)

## Analyst provider production gate

Require sample payloads and documented historical PIT estimates, stable contributor
identity (analyst versus house/team), publication precision/timezone/dissemination,
event order and identity, corrections/retractions, withdrawals, basis semantics,
EPS share basis, currency/unit/scope, historical coverage/gaps and actual-reporting
coverage. Test the entire provider → raw → feature path against independent examples.
Confirm licensing, retention, derived-data use, backups, AI processing and redistribution.
Unknown rights remain unknown. Today's consensus cannot reconstruct past revisions.

## Replaceable connectors

All providers map into canonical contracts with source profiles, immutable raw
receipts, versioned normalization, identities, availability, correction semantics,
units, coverage and rights. Vendor-specific fields stop at adapters. Replacement
requires a new build and cross-provider reconciliation; it must not overwrite old
predictions. Estimate profiles already model much of this; legacy market/macro
storage needs future evolution to meet the same standard.

Institutional market/estimate feeds, transcripts, options, short interest, insider
transactions, institutional ownership, alternative data, supply chains and industry
datasets are future candidates. None is automatically PIT. Publication lag,
restatements, sampling bias and license restrictions differ by category.
See [provider risk register](DATA_PROVIDER_RISK_REGISTER.md).

## Foundation remediation: enforceable contract

The earlier review above describes the starting checkpoint. The remediation adds
bitemporal membership, canonical macro vintages, verified content-addressed raw
storage, immutable recalculation and build identity. See
[implementation audit](FOUNDATION_REMEDIATION_AUDIT.md) for remaining gaps.

**DEVELOPMENT PROVIDER** means data may support connector tests, exploratory
current research and prospective receipt archives, subject to applicable rights.
It does not authorize institutional historical performance claims.
**PRODUCTION-QUALIFIED PROVIDER** means the precise field/endpoint/version and
coverage interval have passed independent timing, revision, identifier, adjustment,
completeness and licensing checks. No existing live adapter has that complete
qualification. Original EDGAR content can be authoritative without certifying the
current aggregate-based fundamental extraction path.

`research/source_policy.py` implements AUTHORITATIVE_PIT, PIT_QUALIFIED,
ARCHIVED_AS_OBSERVED, CURRENT_ONLY, HISTORICAL_AS_SEEN_TODAY and UNQUALIFIED.
`require_historical_replay` accepts only the first two with explicit version,
evidence and coverage at T. Synthetic qualification additionally requires an
explicit test opt-in. `ResearchBuildManifest.require_historical_replay` requires
a matching qualification for every pinned provider/version. No fallback or silent
quality downgrade occurs. Prospective archives can later earn scoped PIT_QUALIFIED
status after validation; merely accumulating files does not grant it.
Future backtest entry points MUST invoke this policy over every required source
and the complete lookback coverage, not merely the endpoint date. There is no
backtester in this checkpoint. Legacy calculators remain development interfaces.

The final qualification checkpoint extends this boundary through the canonical
[`provider_qualifications.yaml`](../config/provider_qualifications.yaml). Runtime
capabilities now include current research, historical replay, raw reproducibility,
PIT actions, historical universe/security coverage, delisted/lifecycle support,
survivorship safety, macro vintages, SEC filing PIT, and verified licensing. A
survivorship request lacking any required capability returns
`BACKTEST_DATA_NOT_SURVIVORSHIP_QUALIFIED`. A development waiver requires named
requester, reason, and reference, appears in returned provenance, and fixes the run
quality to `DEVELOPMENT`. See the
[provider records](provider-qualifications/README.md).

Raw lineage is independently gated as `VERIFIED_ORIGINAL`,
`VERIFIED_RECONSTRUCTED`, `MISSING`, `HASH_MISMATCH`, or
`UNVERIFIABLE_LEGACY`. A refetch is a new `RECONSTRUCTED_COPY` with its own hash,
time, source, and link to the missing record; it cannot certify prior bytes.

SEC fact selection now uses exact aware acceptance evidence when present. A
date-only filing is conservatively unavailable until the next New York midnight.
Company Facts `fy`/`fp` are also stored as reporting-filing context. Later
comparative and amended records retain their own availability and cannot rewrite
earlier knowledge. Company Facts remains current-research-only in the catalog until
broad accession/raw reconciliation is qualified.

### Market field contract

| Field | Stored meaning now | Retrospective change risk | Allowed use |
| --- | --- | --- | --- |
| open | Yahoo chart quote.open, provider-reported; raw-as-traded NOT established | Split normalization/corrections may change history | Development only |
| high | quote.high, same unqualified basis | Same | Development only |
| low | quote.low, same unqualified basis | Same | Development only |
| close | quote.close; never assumed unadjusted merely from its name | Split normalization/corrections; possible double adjustment in raw engine | Development only |
| volume | quote.volume; original share basis not certified | Split/volume corrections or normalization; must reconcile with price basis | Development only |
| adjusted close | indicators.adjclose[0].adjclose, preserved separately by yahoo_chart_v2 | Retrospective split/distribution adjustment; not a historical vintage | Development diagnostics; never an engine input |
| splits | No qualified live canonical action ingestion; archived provider payload only if supplied | Ratios/effective dates/corrections lack known-at archive | Synthetic action tests only |
| dividends | No qualified live canonical action ingestion | Ex/pay/announcement dates, revisions and share basis differ | Synthetic action tests only |
| other corporate actions | Schema supports relationships; no qualified terminal-return feed | Mergers, spinoffs, identifier changes, delisting settlements | Unsupported for production return history |

We cannot positively certify any Yahoo OHLCV field as historically raw. Canonical
rows carry price_basis/volume_basis=PROVIDER_REPORTED_UNQUALIFIED, a normalization
version and original raw receipt. Legacy rows remain UNQUALIFIED. Current Yahoo
requests do not establish complete action retrieval. A raw payload is original
response bytes; that does **not** mean its economic prices are unadjusted.

The frozen Phase 2.4 path remains close + A<=T actions → split-normalized close
and reciprocal volume → TRI. Actions apply economically using effective dates;
announced splits can define a consistent normalization basis even before execution.
Dividends enter on their effective session. available_at null falls back to
creation time, conservatively blocking earlier replay. No formula changed.
Split/reverse split/cash dividend arithmetic is tested. Dividend amounts must
already match the normalized price share basis; multiple later splits are not
production-qualified. Ticker-change/merger/acquisition/spinoff/security-change
records do not implement return conversion, successor stitching or terminal
proceeds. Do not infer delisting returns from missing prices.

A replacement provider must supply raw historical OHLCV and its share basis;
versioned action history; publication/correction timing and historical snapshots;
stable security identifiers with ticker crosswalks; failed/delisted history and
terminal proceeds; documented gaps; retention, backup and usage/redistribution
rights. Its adapter maps to canonical contracts. The legacy MarketPrice table
still keeps first observations per security/time/interval; a qualified replacement
needs versioned provider/build-aware prices before admission, without rewriting
market_v1 formulas. Old outputs require a new build after any changed input basis.

Primary-source recheck: FRED documents today as its default real-time view and
inclusive realtime intervals ([FRED real-time periods](https://fred.stlouisfed.org/docs/api/fred/realtime_period.html)).
Its observations API exposes realtime ranges, vintage_dates, output modes and
pagination ([observations contract](https://fred.stlouisfed.org/docs/api/fred/series_observations.html)).
The [yfinance implementation](https://github.com/ranaroussi/yfinance/blob/main/yfinance/utils.py)
separately parses quote/adjclose and provides adjustment routines; it is not an
upstream historical-raw guarantee. Yahoo's [adjusted-close help](https://help.yahoo.com/kb/SLN28256.html)
was inaccessible during this recheck; no unverified endpoint guarantee is inferred.

### Macro vintage contract

The old EconomicObservation table and FREDConnector remain latest-view context,
not qualified backtest inputs. EconomicVintage and SyntheticMacroVintageConnector
provide a separate, tested canonical foundation. Select source + dataset + series
+ observation date; restrict available_at<=T; then select latest realtime_start
and knowledge version. Missing latest releases remain missing. Receipt is separate
from publication. Same-identity conflicting values fail; new builds preserve old
artifacts. Raw bytes, exact Decimal, timing policy and normalization are retained.

The live ALFRED adapter now requests explicit realtime/observation bounds,
`output_type=1`, and validates pagination/count completeness while preserving raw
pages, revision dates and exact values. A vintage date is not an intraday release
timestamp, so normalization uses a conservative next-day boundary. Initial/revised
mock fixtures pass. Credentialed representative-series coverage, release timing,
missing/delayed/corrected real references and rights still require qualification;
therefore no live historical macro admission is claimed.
