# Alternative data roadmap

Status: FUTURE DESIGN, 2026-09-21. No feeds purchased, scraped, ingested or qualified.
The following are preliminary research hypotheses and engineering estimates, not
verified vendor capabilities, prices or predictive results. Every named source
category is UNQUALIFIED until a specific contract passes the gates below.

PIT requires the exact version that could have been known at T, including panel,
entity mapping, collection latency and later revisions. An old observation date
in a newly reconstructed dataset is insufficient. Historical coverage and costs
below are expectations to test; contractual coverage, price and rights remain unknown.
L/M/H denote relative acquisition/engineering burden, not dollar quotes.

## Candidate assessment

| Source | Investment rationale / likely usefulness | PIT availability requirement | Historical coverage to verify | Noise / confounders | Cost estimate | Licensing / usage risk | Company coverage | Difficulty |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Job postings | Hiring intent by function/geography; contextual early expansion clue | First-seen/removed times and archived versions | Deleted/reposted vacancies and archive start | Evergreen ads, replacement hiring, duplicated recruiters | L–M | Site/feed automation and retention rights unknown | Publicly recruiting firms; subsidiary mapping | M |
| Employee growth | Capacity/cost trajectory; supporting rather than direct demand evidence | Original disclosure/panel vintage, publication lag | Historical panels and employer classifications | Profile lag, acquisitions, outsourcing | M–H | Platform/privacy and aggregate-use rights unknown | Uneven workforce/platform coverage | H |
| Web traffic | Demand funnel for digital businesses; useful with conversion evidence | Collection window, panel vintage and release time | Panel changes and revised historical estimates | Bots, campaigns, geography, traffic without sales | M–H | Licensed panel/derived-use rights unknown | Websites with sufficient measurable traffic | H |
| App downloads / usage | Adoption and engagement; useful for app-dependent products | Store/panel observation and publication versions | Platform/country history, app-ID changes | Paid installs, bots, installs not retention/revenue | M–H | Store/panel and AI-processing rights unknown | App-centric businesses, not whole-market coverage | H |
| Developer activity | Ecosystem adoption; potential technical-product corroboration | Archived release/package/contribution observations | Package/organization history and deletions | Automation, hobby use, dependency downloads | L–M | Registry/API terms and content rights unknown | Developer-facing products | M |
| GitHub activity | Public engineering/ecosystem proxy; limited revenue inference | Event receipt/version plus PIT company/repository mapping | Renames, transfers, deletions and event retention | Bots, mirrors, private work missing, stars purchased | L–M | API terms differ from repository/content licenses; verify | Open-source-visible firms only | M |
| Product pricing | Realized or advertised pricing power/inflation; useful when scope matches | Dated SKU/channel/region quotes and revisions | Promotions, discontinued SKUs, prior configurations | List vs transaction price, mix, currency and taxes | M | Collection/reuse permissions unknown | Observable catalogs or licensed transaction panels | H |
| Product availability | Supply/demand imbalance clue; combine with inventory context | First-seen inventory status by channel/time | Historical stock-outs and site changes | Logistics failure, artificial scarcity, site outages | M | Site/feed extraction rights unknown | Retail/channel-visible products | M–H |
| Search trends | Attention or intent; weak alone | Query/window parameters and saved returned vintage | Sampling, rescaling and regional methodology | News curiosity, ambiguous brand terms, seasonal searches | L–M | API/export/reuse rights unknown | Consumer brands and searchable topics | M |
| Patent filings | Innovation direction and IP exposure; slow strategic context | Public disclosure availability, not private filing date alone | Publication, assignment, family/status history | Quantity vs value, long lags, strategic defensive filings | L–M | Office/provider access and enrichment rights to verify | Patent-intensive sectors | H |
| Government contracts | Potential backlog/customer evidence; award not recognized revenue | Award announcement/availability and amendment versions | Cancelled awards, obligations and modifications | Ceilings vs funded amounts, subcontract attribution | L–M | Jurisdiction/source terms and derived use unknown | Government-exposed contractors | M–H |
| Imports/exports | Product/geography shipment exposure; sector-dependent proxy | Release vintage and customs/reporting lag | Revisions, commodity codes and entity mappings | Aggregation, transfer pricing, rerouting, inventories | M–H | Public/commercial source rights differ; unknown | Trade-exposed firms, often indirect mapping | H |
| Shipping | Physical activity and supply-chain corroboration | Position/manifest receipt, publication and corrections | Vessel coverage gaps and historical ownership | Cargo attribution, transshipment, inventory build | M–H | Tracking/manifest retention and use rights unknown | Physical goods/transport sectors | H |
| Satellite data | Facility utilization/construction proxy; niche potential | Image capture AND delivery/processing/model vintage | Cloud gaps, cadence, sensor/model changes | Weather, ambiguous activity, site-to-company attribution | H | Imagery/derived-product/AI rights unknown | Observable fixed facilities | H |
| Credit-card aggregates | Consumer spending direction; potentially useful in covered segments | Transaction/report delay, panel and merchant-map vintage | Backfills, panel churn, refunds and revisions | Demographic bias, cash/B2B absent, processor changes | H | Privacy, consent, retention and aggregate-use rights unverified | Panel-covered consumer merchants | H |
| Customer reviews | Product problems/satisfaction; qualitative corroboration | Original posted/edited/received versions | Removed reviews and changed rating scales | Fake reviews, selection bias, moderation, small samples | L–M | Platform/content/privacy permissions unknown | Consumer-facing products/services | M–H |
| Channel checks | Inventory/pricing/customer context; narrow but potentially direct | Interview/observation time, receipt and usable consent | Usually sparse prospective notes; history unverified | Selection, anecdote, leading questions and incentives | H | Consent, confidentiality and information eligibility require review | Selected channels/products | H |
| Supplier activity | Capacity/orders corroborating customer demand | Publication/receipt and versioned supplier/customer links | Relationship changes and disclosure history | Multi-customer exposure, double counting, inventory cycle | M–H | Provider/disclosure/relationship-data rights unknown | Disclosing or observable supply chains | H |

## Qualification contract

For each candidate capture provider/profile and dataset/version IDs, raw hashes,
collection/event/period/publication/receipt/availability timestamps, timezone and
precision, corrections, population/sample/panel definitions, geography, metric units,
methodology/model version, stable entity/SKU/site mappings and their knowledge time,
coverage/gaps, license reference and permitted uses. Distinguish measured observations
from vendor estimates and internal inferences. Saved raw bytes are necessary where
permitted, but do not themselves establish historical availability.

A panel reconstructed today needs documented old vintages before historical replay.
Changing membership/coverage can create false growth; record denominators, matched
panels and sensitivity to panel churn. Today's company ownership must not relabel a
past supplier or repository. Null coverage is not zero activity. Vendor backtests
are not independent proof for this system.

Evaluate source lineage and economic independence: a vendor feature derived from
management releases is not fresh corroboration. Avoid raw counts as quality labels:
more jobs, commits, patents or reviews do not automatically mean a better business.
Use [evidence hierarchy](RESEARCH_EVIDENCE_HIERARCHY.md) and
[corroboration design](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md#disagreement-and-corroboration).

## Recommended research order, not purchase authorization

1. After foundation remediation, prototype small synthetic evidence contracts and
   legally usable original disclosures. Prioritize documented pricing, public
   contract announcements and dated product/customer events with clear mechanisms.
2. Evaluate a limited prospective archive for permitted jobs/developer/product
   evidence. Measure entity precision, stability, timeliness and cost before breadth.
3. Consider licensed traffic/app/trade/supplier panels only after obtaining samples,
   rights and historical-version proof. Compare against simple existing features.
4. Defer satellite, card panels and extensive channel checks until a specific
   hypothesis and economic benefit justify their cost and governance burden.

These priorities reflect expected interpretability and effort, not established alpha.
Do not collect personal employee/customer profiles, confidential channel information
or nonpublic transaction records by default. Evaluate lawful aggregate sources and
source-specific permissions; do not infer consent or authorized use from accessibility.

## Validation and exit criteria

Register hypothesis, target companies, objective/horizon, expected lag, coverage,
source budget and minimum useful incremental performance before looking at outcomes.
Test proxy-to-business linkage first, then incremental predictive value using
chronological, mature-label, broad-universe comparisons and held-out periods.
Ablate each family, control size/sector/coverage and correlated origins, include
failures and unchanged firms. Log every candidate tried to avoid multiple-testing bias.

Reject or restrict a feed with unprovable timing, unstable mappings, unusable rights,
large unexplained revisions, narrow biased coverage, or no reproducible incremental
value. Persist the rejection and costs. Monitor panel/feature/provider drift after
qualification; feed failure opens a research/data incident, not automatic production
formula changes. No source in this roadmap is production-approved by this document.
