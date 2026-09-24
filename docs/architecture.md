# System architecture

Current strategic verdict: **CONDITIONAL PASS**. Phase 2.5 is implementation
complete, independently unfrozen; Phase 2.6 has not started. See the
[complete blueprint](INVESTMENT_RESEARCH_SYSTEM_BLUEPRINT.md) and
[strategic review](STRATEGIC_ARCHITECTURE_REVIEW.md) for evidence and gates.

```text
Connectors → archived raw receipts → canonical normalization → PIT evidence
→ features → factors → signals → frozen research snapshot → prediction ledger
→ future outcomes → outcome ledger → backtest → calibration → challenger
→ validation → versioned promotion
```

Features exist for fundamentals, markets and estimates. Factors onward are future
work. AI will synthesize cited snapshots, not create metrics or secretly rank stocks.
Production cannot silently modify itself; every change passes a versioned gate.

Connectors are intentionally replaceable. Each maps vendor payloads to canonical
identity, units/basis, availability, correction, coverage and provenance contracts.
Future institutional feeds, transcripts, options, ownership, short interest,
insider transactions, alternative/supply-chain/industry data require qualification,
not provider-specific logic embedded in research formulas. Replacement creates a
new source profile/build and preserves old evidence. Existing legacy schemas need
further versioning to fully enforce this boundary.

PostgreSQL is the structured system of record; raw payloads currently use local
files. Parquet/DuckDB are possible analytical extensions, not an implemented
backtester or required new infrastructure. Off-host object storage and restore
drills are planned in [durability](DATA_DURABILITY_AND_RECOVERY.md).

Timestamp fields alone do not establish PIT correctness, and files with hashes
alone do not guarantee immutability. Historical universe knowledge, SEC filing
visibility, provider adjustment/vintage semantics, generic raw path collisions
and mutable legacy feature publication are documented gaps. Estimate builds and
published states have stronger immutability checks. See
[source trust](DATA_SOURCE_TRUST_AND_PIT.md) before extending a source or making
historical performance claims.

## Future company-level research boundary

[BusinessQualityResearch and CompanyStrategicEvents](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md)
add scoped dossiers and versioned events beside structured numerical features.
Professional research artifacts and alternative observations use the same canonical
PIT evidence/permission contracts. [Source classes](RESEARCH_EVIDENCE_HIERARCHY.md)
do not automatically establish correctness or independence.

Claims link to corroborating/contradicting evidence and preserved external disagreements,
then to a frozen thesis graph and individual outcome labels. Business-quality
assumptions feed future scenario valuation; AI only explains the packet. No opaque
combined score, moat-rating implementation, scraping or new phase is introduced.
The [alternative roadmap](ALTERNATIVE_DATA_ROADMAP.md) describes unqualified future
candidates; current remediation gates remain prerequisites.
