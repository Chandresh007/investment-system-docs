# Research-platform comparison, valuation and business quality

Status: conceptual roadmap; no proprietary reports copied and no new valuation engine.

Morningstar's public materials emphasize intrinsic value, competitive advantage,
uncertainty and capital allocation. These concepts help distinguish an improving
business from an attractive investment at its current price.
[Morningstar investing guide](https://www.morningstar.com/stocks/morningstars-guide-investing-stocks)
Its report interface also identifies business strategy, valuation/profit drivers,
risk and capital-allocation sections.
[Morningstar research reports](https://developer-beta.morningstar.com/direct-web-services/documentation/data-and-research/research-reports)

The comparison below is our design assessment, not a claim that a commercial
platform lacks quantitative tools or that this unfinished project outperforms it.

| Research dimension | Morningstar-style company report | Current project and target |
| --- | --- | --- |
| Business strategy/outlook | Analyst interpretation of business economics | Structured fundamental evidence exists; qualitative synthesis absent |
| Bull/bear cases | Competing plausible narratives | Future explicit claims, evidence and invalidation ledger |
| Economic moat | Durability of competitive advantage | Future evidence-based quality assessment |
| Fair value/profit drivers | Intrinsic valuation tied to assumptions | No valuation engine; planned scenarios and sensitivity |
| Financial strength/risk/uncertainty | Balance-sheet resilience and uncertainty about outcomes | Existing cash-flow, volatility and drawdown features; incomplete leverage/solvency analysis |
| Management/capital allocation | Judgment about execution and reinvestment | Future sourced history; no unsupported management score |
| Broad discovery | Company coverage and research products | Target automated scanning across thousands; not scale-qualified today |
| Reproducibility | Depends on product/report archive | Local strength: explicit formulas/provenance, particularly estimate replay; legacy PIT gaps remain |
| Empirical learning | Not assessed from public report format | Target frozen predictions/outcomes and controlled validation; not implemented |

The project already offers inspectable feature calculations and strong synthetic
estimate-event tests. Its intended advantage is breadth, early inflection detection,
cross-sectional comparison and reproducible empirical evaluation. Those are target
strengths, not proven investment results. Deep business judgment, intrinsic valuation
and narrative risk analysis are currently weaker or absent. Combine these capabilities
rather than mistaking growth/momentum detection for a complete equity report.

## Serious valuation workstream (future)

Before Phase 2.11's full research release, design and validate:

- DCF/intrinsic value with explicit forecast cash flows, reinvestment, discount rates,
  terminal assumptions, debt/cash and share-count bridge; scenario and sensitivity tables.
- FCF yield; EV/Revenue; EV/EBITDA; P/E only with economically meaningful earnings;
  PEG-style relationships only with declared growth units and valid domains.
- Historical own-valuation bands reconstructed from then-known inputs, and peer
  valuations using PIT membership, comparable accounting and capital structures.
- Growth-adjusted valuation, bear/base/bull scenarios and margin-of-safety ranges,
  preserving assumptions rather than publishing a falsely precise target price.

Negative EBITDA/earnings, banks, insurers, dilution, stock compensation, leases,
cyclical peaks and acquisition effects need explicit applicability rules. Enterprise
and equity cash flows must use consistent discounting and capitalization. Forward
estimates require qualified forecast basis/share identity; no current consensus
backfill. Every fair-value scenario is a versioned judgment with its own horizon
and later diagnostic record, not a canonical observed fact.

A first valuation context may precede factors; a mature engine can develop alongside
2.7–2.10. Missing valuation means “research candidate, valuation not assessed,” not
an attractive entry recommendation. No formulas from frozen phases are modified.

## Qualitative business-quality workstream (future)

| Question | Evidence to preserve |
| --- | --- |
| Economic moat, switching costs | Customer retention, integration burden, contracts and sourced case evidence |
| Network effects | Usage/network economics; distinguish asserted effects from measured ones |
| Pricing power, cost/scale advantage | Realized price/mix, unit cost and peer context |
| Customer/supplier concentration | Disclosed exposure and supplier relationships, with dates and denominator |
| Management execution/capital allocation | Prior commitments versus delivered results, reinvestment/acquisitions/buybacks |
| R&D productivity | Product outcomes and investment history, without inventing a universal ratio |
| Competitive intensity | Competitor actions, entry barriers and margin pressure |
| TAM, market share, product leadership | Named market definition, vintage size estimates and independent supporting sources |

Combine structured evidence with later AI synthesis. AI must cite the precise
source version, distinguish observation from interpretation, display missing and
conflicting evidence, and never invent metrics. A plausible moat narrative is not
proof of durable returns; validate claims across failures as well as compounders.

## Public methodology study and concrete extension — 2026-09-21

The [qualitative architecture](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md)
now documents the public methodology/help sources, BusinessQualityResearch,
evidence-backed moat hypotheses, CompanyStrategicEvents, professional-research
contract, corroboration and disagreement handling. It separates our proposed
controls from Morningstar's intellectual framework and does not clone ratings.

Business quality and competitive durability inform explicit valuation assumptions;
current price and uncertainty remain separate. Management forecasts are attributed
claims to evaluate, not automatically verified facts. External reports may disagree
with our system and remain visible without forcing consensus.

Morningstar methodology may be studied. Legitimately owner-supplied subscriber
reports may be reviewed for the owner's research subject to applicable permissions.
Systematic data requires verified licensing/API rights; no unauthorized automated
scraping, redistribution or assumed AI/training rights. All new layers remain designs.
