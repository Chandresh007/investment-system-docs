# Research evidence hierarchy

Status: FUTURE DESIGN, 2026-09-21. No ingestion or scoring implemented.
Source classes guide questions and provenance; they do not automatically determine
correctness. An official presentation proves what management said, not that its
forecast will occur. A professional analyst's conclusion remains an opinion.

| Class | Examples | What it can establish / what to challenge |
| --- | --- | --- |
| PRIMARY | SEC filings, earnings releases, official presentations, regulatory disclosures, government data | Original disclosure or measured series; challenge accounting scope, revisions, incentives, release timing and forecast assumptions |
| DIRECT_COUNTERPART | Customer, supplier, partner announcements; competitor disclosures | Another party's observation; check commercial incentives, scope and whether it merely repeats the issuer's statement |
| PROFESSIONAL_RESEARCH | Morningstar-style reports, licensed sell-side/industry research, specialist publications | Informed interpretation and external assumptions; not independent ground truth or guaranteed prediction accuracy |
| ALTERNATIVE_DATA | Web traffic, job postings, app usage, developer activity, search trends, pricing, shipping, government contracts, patent activity | Proxies for activity; validate coverage, denominators, sampling and economic connection |
| GENERAL_INFORMATION | News, blogs, forums, social media | Leads, reporting and attributed perspectives; trace originals, verify identities and distinguish hearsay from observation |

Classification applies to an artifact/assertion, not permanently to a brand.
A news article carrying an original interview has different evidence from its
headline commentary. A government contract notice can be PRIMARY evidence of an
award while its aggregate use is an alternative-data feature. Preserve both the
source class and analytical evidence family; do not force them into one field.

## Evidence record contract

Every eligible assertion references an immutable evidence version containing:

- Evidence ID, source/provider, source class and analytical family, stable company/
  entity mapping with its own timing, document/version ID, URL and raw provenance.
- Observation/event/period date, published_at, available_at, retrieved_at, timezone,
  precision, availability basis, correction/supersession links and build ID.
- Exact locator (page/section/table/cell/span), permitted short excerpt or restricted
  reference, content hash, unit/basis/population when quantitative, and extraction version.
- Statement type: OBSERVED, REPORTED_CLAIM, FORECAST, INTERPRETATION or SCENARIO.
  An extracted forecast is never silently converted into an observed fact.
- Source-quality assessment and rubric version: directness, identity assurance,
  coverage, timeliness, correction history, incentives/conflicts and limitations.
- Separate extraction confidence, entity-resolution confidence and claim-support
  confidence, each with reasons; unknown is explicit. None implies return probability.
- Corroboration/contradiction links, common-origin/dependency group, reviewer/author,
  review status, as-of timestamp, rights classification and permitted-use record.

The hierarchy does not assign automatic numeric weights. Confidence is initially a
reasoned categorical assessment (LOW/MEDIUM/HIGH/UNKNOWN), not a fabricated probability.
A later calibrated rubric requires versions, held-out evaluation and promotion.

## Independence, timing and disagreement

Ten articles repeating one press release are one underlying evidence origin.
Record `derived_from` links and `independence = VERIFIED / RELATED / UNKNOWN`.
An analyst citing the same management claim is not independent corroboration.
Absence of contradiction is not proof of support. Missing source coverage is not
negative business evidence. Conflicts stay visible and may produce DISPUTED rather
than a majority vote or forced consensus.

At research T select only versions with justified available_at <= T. The complete
chain—including mappings, supporting documents and rubric—must be eligible.
Prospective operational knowledge cannot precede receipt. Historical reconstruction
may use a later import only with certified dissemination for that exact historical
version and an explicitly separate build. A dated report regenerated today is not
proof of old wording. Preserve corrections and old conclusions.

Licensed/proprietary evidence stays in authorized storage, never public Git by
default. Reading permission does not establish bulk extraction, AI processing,
redistribution or training rights. Record source-specific permissions rather than
inferring them from this hierarchy. See [professional contract and use policy](QUALITATIVE_AND_ALTERNATIVE_RESEARCH_ARCHITECTURE.md#professional-research-contract-and-use-policy).

## Future acceptance examples

Reject a present-day revised report in an old snapshot; keep a management forecast
as a claim; deduplicate syndicated support; preserve a supplier contradiction;
mark an unresolved ticker mapping unknown; show a missing panel without assigning
zero demand; and trace each qualitative assertion to the permitted source version.
Use small synthetic/legally usable cases, then held-out documents and human review.
