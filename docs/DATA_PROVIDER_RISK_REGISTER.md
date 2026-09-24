# Data provider risk register

2026-09-22; living qualification register, not procurement approval. “Preferred”
means required capabilities, not an endorsed or purchased vendor. Rights and SLAs
are unknown unless a contract is actually verified. Primary evidence and repository
paths are in [Data trust and PIT](DATA_SOURCE_TRUST_AND_PIT.md).

| Provider / category | Current purpose | PIT quality | Revision risk | Licensing / usage risk | Availability reliability | Replacement difficulty | Future preferred source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| SEC EDGAR filings | Fundamental evidence / planned reporting projection | Authoritative original content; timing/extraction conditional | Amendments and dissemination timing | Public access; access policy applies; third-party exhibit rights not certified | Public service, no project SLA established | Medium: accession/context mapping | Original accession archive with verified availability |
| SEC Company Facts | Implemented actuals normalization | Historical-as-seen-today; accession qualification needed | Comparative facts, restatements, extraction changes | API access available; downstream rights review still needed | API lag/coverage must be measured | Medium/high: tag and fiscal semantics | Original filings plus qualified extraction and append-only versions |
| FRED | Basic macro ingestion | Latest view, not vintage replay | Revised GDP/employment and other releases | Series-specific third-party restrictions unverified | No project SLA established | Medium: schema and vintage identity change | ALFRED/realtime plus original release evidence |
| ALFRED / FRED vintages | Canonical vintage store plus explicit live adapter; qualification pending | PIT if correctly selected | Vintage coverage and date-only boundaries | Series and retention rights unverified | Coverage varies; measure retrieval completeness | Medium | Qualified vintage archive, release timestamps |
| Yahoo chart | Development OHLCV | Unqualified raw PIT basis; retrieval-gated | Retrospective adjustments/corrections | Commercial use, storage and redistribution rights unverified | No contracted SLA; endpoint/rate-limit risk | Medium/high: canonical price contract remains, schema needs versioning | Licensed raw prices, actions, corrections and delisted history |
| yfinance | Not used by current connector | No improvement to upstream PIT | Library options/repair plus upstream revisions | Software license is not a data license | Unaffiliated wrapper; no project SLA | Low adapter, high qualification work | Same qualified market contract |
| RSS publishers / feeds | Basic news ingestion | Archive-ourselves; no certified historical replay | Edits, deletion, syndication and incorrect dates | Full text, storage and AI processing rights unknown per source | Feeds can truncate/disappear | Medium: entity and version semantics | Licensed timestamped archives plus original releases |
| Synthetic estimates | Tests only | Explicit synthetic profile | Controlled fixtures | Synthetic fixtures suitable for regression; not market data | Deterministic local tests | Low | Retain as adapter acceptance suite |
| Future estimate vendor | No production feed | Unqualified until event-history proof | Corrections, withdrawals, contributor changes, basis changes | License/retention/derived use/AI rights unknown | Contract and historical gaps unverified | High semantic qualification, bounded by canonical profile | Licensed contributor event archive with complete coverage evidence |
| Corporate-action vendor | Schema and synthetic fixtures only | Unqualified production history | Effective dates, revisions, split/dividend basis | Retention/redistribution unknown | Completeness unproven | High reconciliation cost | Issuer/exchange evidence plus qualified historical action feed |
| Universe / classification provider | Configured subset and bitemporal evidence schema | Effective AND knowledge time implemented; real history unqualified | Reclassification, ticker reuse, delisting omissions | Taxonomy and history licensing unknown | Broad coverage unproven | High historical reconstruction cost | Bitemporal security master with failed/delisted names |
| Institutional market feed | Planned replacement/augmentation | Unqualified until contract test | Corrections and adjustment modes | Unknown | Unknown | Medium after versioned contracts | Provider demonstrating required coverage and rights |
| Transcripts / earnings releases | Planned | Unqualified; archive publication versions | Edited transcripts, speaker corrections | Text/AI/retention rights unknown | Unknown | Medium | Original releases plus licensed versioned transcripts |
| Options | Planned | Unqualified | Corrected quotes, survivorship, stale chains | Unknown exchange/vendor rights | Unknown | High contract/surface normalization | Timestamped historical chains and contract master |
| Short interest | Planned | Unqualified | Settlement versus publication dates, revisions | Unknown | Unknown | Medium | Publication-vintage archive |
| Insider transactions | Planned | PIT if original filing and amendments preserved | Late reports/amendments | Source-specific rights unverified | Unknown | Medium | Original filing evidence and transaction/report distinction |
| Institutional ownership | Planned | Unqualified current holdings; filing PIT possible | Reporting lag, amendments, changing identifiers | Source-specific rights unverified | Unknown | Medium/high | Filing-vintage holdings, no period-end backdating |
| Alternative data | Planned | Unqualified | Panel changes, historical backfill, sampling bias | Collection/usage/retention rights unknown | Unknown | High | Auditable historical panels and legal provenance |
| Supply-chain data | Planned | Unqualified | Revised supplier/customer links and estimates | Unknown | Unknown | High | Versioned relationship evidence with known timing |
| Industry datasets | Planned | Unqualified | Revised market sizes/share and taxonomies | Unknown | Unknown | High | Vintage observations and historical classifications |
| Official macro/policy/event sources | Planned factual event context | PIT if original publication/version archived | Corrections and effective-date changes | Source-specific rights unknown | Unknown | Medium | Original timestamped announcements and documents |

Before qualification, record accountable owner, provider profile/version, trial
sample hashes, coverage by year/sector/size, timestamp precision, known gaps,
licensing evidence reference, permitted uses, retention constraints, and review date.
Track decisions as accepted/restricted/rejected with reasons; do not elevate a
source simply because an adapter passes synthetic tests. No new licenses were
purchased or inferred in this review.


Historical remediation checkpoint: content-addressed raw storage, PIT membership readers,
macro vintage schema, market basis labels, immutable feature recalculation, source
admission policy and local restore tooling are implemented. These close code defects,
not provider qualification. Yahoo remains development-only; no live ALFRED or
production estimate qualification exists. Off-host recovery and persistent raw
inventory reconciliation were still open at that checkpoint. See
[remediation audit](FOUNDATION_REMEDIATION_AUDIT.md).

Final qualification update: the machine-readable
[`provider_qualifications.yaml`](../config/provider_qualifications.yaml) is now the
runtime authority, with explanatory records in
[`provider-qualifications/`](provider-qualifications/README.md). Yahoo and ordinary
FRED remain development/current only. SEC original accessions are conditional for
their exact archived scope; Company Facts remains current-only for runtime replay
admission. The ALFRED connector exists but is
`LIVE_ALFRED_CONNECTOR_PENDING`. No live source has the combined universe,
historical-security, action, delisting, lifecycle, rights, and survivorship
capabilities. Off-host upload tooling is ready, but live cloud qualification remains
external. The earlier “open” items are now hard-gated rather than silently usable.
