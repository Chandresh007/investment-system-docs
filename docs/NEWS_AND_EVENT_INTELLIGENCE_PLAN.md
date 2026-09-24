# News and event intelligence plan

Status: FUTURE DESIGN. Basic RSS ingestion and article/entity models exist;
production deterministic event intelligence does not. Existing ticker-based entity
links and publication parsing are starting points, not qualified event truth.

## Canonical event layer

```text
source receipt → immutable document version → qualified timing/entity resolution
→ typed event assertion → validation → published event version → research evidence
```

Each event preserves event ID and version, source/URL/raw hash/record locator,
`published_at`, `available_at`, `retrieved_at`, event/effective time and timezone
precision, entities with stable IDs, event type, economic direction where evidenced,
confidence in extraction/entity matching, evidence spans, extraction/rule version,
coverage, status and correction/supersession links. Confidence describes evidence
quality; it is not a stock-return probability. Preserve original currencies/units.

Deduplicate syndication while retaining distinct confirming sources. Distinguish
rumor, company announcement, independent confirmation and completed event.
An announced acquisition is not a closed acquisition. Corrections append versions;
a later edited article cannot appear in an earlier snapshot. Uncertified RSS
publication dates use receipt-based availability. Unknown timezone stays uncertain.

## Company event taxonomy

| Categories | Typical deterministic checks |
| --- | --- |
| Earnings, guidance | Absolute fiscal target, actual/forecast basis, release time |
| Customer wins, product launches | Named parties, contract versus rumor, announcement versus delivery |
| Capacity expansion | Location, size/unit when disclosed, planned versus operational |
| M&A, management changes | Parties, roles, announcement/effective/completion distinction |
| Litigation, regulation | Official document, procedural status, applicable entities and dates |
| Supply chain, competitor actions, pricing changes | Exposure link and sourced mechanism, not headline sentiment alone |
| Security incidents | Confirmed affected entity, disclosure time and scope uncertainty |

Start with deterministic source-specific parsers and validation rules. LLMs may
propose structured extractions with exact evidence spans, but schema checks,
identity/time rules and evidence verification govern publication. Persist accepted
extractions and model/prompt versions so replay does not require a fresh stochastic
answer. LLM sentiment cannot be the sole source of truth or a hidden ranking score.

## Macro, geopolitical and policy context

Future taxonomy includes rates, inflation, employment, credit and liquidity;
trade restrictions, tariffs, sanctions and export controls; conflicts, shipping
disruptions and regulatory changes; elections, legislation, tax changes and
industrial policy. This is a data schema plan, not commentary on any current event.

Record factual source documents, announcement/effective dates, jurisdiction,
affected activities, legal status and later corrections. Preserve announced,
proposed, enacted, implemented, stayed and repealed states when applicable.
Economic-series inputs require vintage-aware macro data, not latest FRED history.

For each exposed company/industry, record a separate evidence-backed relation:
exposure type, mechanism of impact, known timing, data gaps and scenario uncertainty.
Examples of mechanisms include funding cost, input cost, customer demand, shipping
routes, permitted exports or tax treatment. Exposure relationships also need PIT
versions; today's supply chain cannot silently explain a past decision.

Treat political and election information neutrally as factual market context.
Do not score or rank politicians, parties, policies or legislation, express political
desirability, or forecast election outcomes. Research may explain conditional
company exposure to documented events without turning that into a political judgment.

## Thesis integration and validation

A thesis references event/assertion IDs, expected evidence and a deadline. Later
checks return occurred/not occurred/unknown, supported by qualified evidence;
absence of a feed item alone cannot establish that a catalyst failed. Keep
extraction correctness, economic interpretation and investment outcome separate.

Acceptance cases: duplicated headlines, revised article at the same URL, late
receipt, future publication timestamp, ticker reuse, ambiguous customer, guidance
withdrawal, planned capacity not completed, conflicting sources, and a correction
crossing research T. Evaluate precision/recall on legally usable, independently
annotated documents and unseen publishers/time periods. Report source coverage.

Basic taxonomy and ledger references are dependencies for deeper snapshots;
production breadth belongs to proposed Phase 2.12. No live collection or AI event
extraction was implemented in this review.

## CompanyStrategicEvents extension

The [qualitative research architecture](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md#companystrategicevents-what-is-happening-now)
extends this same event layer with products/generations, customer/design wins,
capacity/factories, geographies, partnerships, pricing/distribution, acquisitions,
divestitures, R&D, management/restructuring, debt, repurchases, customer losses,
supplier problems and competitive launches. There is no second incompatible event store.

Preserve company/roles, event_type, event_date, published_at, available_at, source,
evidence, estimated mechanism and confidence, plus version/receipt/lifecycle metadata.
Announcement, execution and economic impact are separate assertions. Link events
to typed thesis catalysts/risks; preserve contradictory sources and original versions.
Management claims require corroboration, not automatic promotion to verified facts.
No implementation or production event scores are introduced by this extension.
