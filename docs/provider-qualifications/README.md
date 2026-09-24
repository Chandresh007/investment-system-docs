# Provider qualification records

These records explain the machine-readable capability catalog in
[`config/provider_qualifications.yaml`](../../config/provider_qualifications.yaml).
The catalog is the runtime source of truth. A document cannot grant a capability
that the catalog withholds. A provider name alone never grants historical replay:
each run must also pin the exact version, evidence reference, coverage interval,
research timestamp, and any synthetic or development override.
Runtime admission rejects catalog-unknown sources and any mismatch in provider
version, trust, qualification version, capability set, or synthetic classification.

Catalog version: `phase-2.6-qualification-v1`, reviewed 2026-09-22. Existing live
provider decisions are unchanged; the new capabilities belong only to the
versioned synthetic survivorship/classification fixture.

| Provider profile | Production status | Historical replay |
| --- | --- | --- |
| [Yahoo market](yahoo-market.md) | Development only | Rejected |
| [SEC EDGAR / Company Facts](sec-edgar.md) | Accession scope conditional; aggregate partial | Only archived accession evidence |
| [FRED](fred.md) | Current context only | Rejected |
| [ALFRED](alfred.md) | `LIVE_ALFRED_CONNECTOR_PENDING` | Rejected until live qualification |
| [Synthetic estimates](synthetic-estimates.md) | Tests only | Explicit synthetic opt-in only |
| [Synthetic survivorship/classification](synthetic-survivorship.md) | Tests only | Explicit synthetic opt-in; always `DEVELOPMENT` |

No cataloged live provider is currently survivorship-qualified or production-
research-qualified. No record establishes commercial usage, redistribution,
retention, backup, or AI-processing rights unless a future version cites the
reviewed agreement.
