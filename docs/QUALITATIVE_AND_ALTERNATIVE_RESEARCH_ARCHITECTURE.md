# Qualitative and alternative research architecture

Status: DOCUMENTATION / FUTURE DESIGN, 2026-09-21. Starting checkpoint
`4c992ca4e2c74773d53ce4135febb3cce56c6deb`, clean master = origin/master.
No production scores, moat ratings, connectors, scraping, tables or financial
formula changes. Phase 2.6 has not started; Phase 2.5 remains independently unfrozen.
The preceding strategic CONDITIONAL PASS and R1–R9 remediation gates still apply.

## Recommendation and public methodology study

Add inspectable company-level research alongside numerical features, with shared
PIT evidence contracts. Distinguish observation, attributed claim, interpretation,
forecast and scenario. Preserve disagreements, including professional views that
challenge our thesis. Qualitative judgment cannot become true merely because its
storage or replay is deterministic.

Morningstar's publicly accessible 2020 methodology describes fundamental analysis
feeding cash-flow valuation; moat assumptions influence the persistence of economic
profits. Estimated value, uncertainty and current price play distinct roles in its
investment assessment. We adopt that separation and scenario discipline, not its
rating algorithm, thresholds or proprietary model. This is a historical methodology
reference, not a claim that all 2020 settings remain current.
[Public methodology PDF](https://advisor.morningstar.com/Enterprise/VTC/MasterEquityResearchMethodology_Oct2020.pdf)

The current public methodology catalog lists analyst equity research separately
from quantitative equity ratings and fund methodologies. This design studies the
analyst framework, not fund stars or an inferred quantitative clone.
[Morningstar methodology catalog](https://www.morningstar.com/business/insights/research/methodology-documents)

The public company-report help describes strategy/outlook, balanced bull/bear
arguments, financial strength, moat, valuation/profit drivers, risk/uncertainty and
capital allocation. Those are useful research questions, not answers to copy.
The help page was available through indexed public text; direct open returned 403.
[Company-report help](https://www.morningstar.com/help-center/reports/company-reports)

Its public capital-allocation explanation examines financing resilience, investment
choices and shareholder distributions. Our management-execution extension will
compare dated commitments with subsequent delivery, separately from charisma or
share-price performance. The explanation was available in indexed public text;
direct retrieval returned 403. [Capital allocation](https://www.morningstar.com/stocks/why-companys-management-capital-allocation-matter)

A March 2026 public moat article describes five structural advantage categories
and links competitive durability to valuation. We use those categories as prompts
for independently evidenced hypotheses, not imported production ratings or fixed
moat-duration assumptions. [Public moat framework](https://www.morningstar.com/business/insights/blog/equity-economic-moat-ratings)

No subscriber report was accessed, copied or archived for this task. Public links
and short original summaries are the research record; publisher materials are not
committed to Git. No Morningstar-specific API/license entitlement was established.

## BusinessQualityResearch

Proposed output: a versioned company research dossier as of T, containing scoped
assertions rather than a scalar quality score. Reuse the canonical evidence IDs,
company/security identities and frozen snapshot manifests. Segment/product/geography
scope matters: a defensible product does not automatically imply a company-wide moat.

| Research dimension | Questions and evidence to preserve |
| --- | --- |
| Business model / product portfolio | Who pays, for what, how often; product generations and segment economics; distinguish recurring from transactional revenue |
| Revenue drivers / customer base / geographies | Units, price, mix, installed base, end markets and regional exposures with period/denominator definitions |
| Competitive landscape / market share / TAM | Named competitors, market definition, measurement vintage, share numerator/denominator; competing TAM estimates remain separate |
| Pricing power | Realized prices, discounting, churn and mix; distinguish a list-price announcement from collected revenue |
| Switching costs / network effects | Integration burden, retention and incremental participant value; distinguish lock-in claims from observed behavior |
| Cost advantage / capital intensity | Comparable unit economics, utilization, reinvestment and capacity cycle; no margin comparison without compatible scope |
| Intangible assets / IP / efficient scale | Rights, expiry, substitutability, entry economics and market boundaries; existence of a patent alone is insufficient |
| Distribution advantage / ecosystem strength | Channel access, partner dependence, platform adoption and barriers; avoid counting the same evidence in several moat categories |
| R&D effectiveness | Dated investment, milestones, product adoption and failures; spending or patent counts alone do not prove productivity |
| Customer / supplier concentration | Exposure percentages where disclosed, contractual dependency, alternatives and counterparty identity |
| Management execution | Original promises, deadlines, scope changes and delivered outcomes; disclose outside constraints and attribution uncertainty |
| Capital allocation | Reinvestment/acquisition assumptions, financing, debt, dilution, buybacks and distributions; actual allocation versus authorization |

Dossier fields: dossier/version/build IDs, company, scope, research T, created_at,
assertion IDs, source and counter-evidence IDs, coverage/missingness, author/reviewer,
rubric version, confidence reasons and prior-version links. No estimate or metric is
fabricated to fill a narrative. Unknown customer concentration stays UNKNOWN.
Historical evolution uses immutable assertions; later understanding creates a new
version, not a retroactive “we always knew” explanation.

## Evidence-backed moat research, without production ratings

Each category assertion stores: company/segment, category, proposed mechanism,
evidence, counter-evidence, source/locator, available_at, as-of T, confidence and
rationale, applicability, expected durability assumptions, reviewer/rubric version,
prior assertion and change reason. All evidence follows the
[evidence hierarchy](RESEARCH_EVIDENCE_HIERARCHY.md).

| Category | Potential supporting evidence | Counter-evidence to seek |
| --- | --- | --- |
| INTANGIBLE_ASSETS | Enforceable IP, demonstrable brand premium or scarce license tied to economics | Expiry, substitutes, adverse rulings, brand discounting |
| SWITCHING_COSTS | Documented migration burden, workflow integration, retention under price increases | Cheap migration tools, falling retention, interoperability or customer exits |
| NETWORK_EFFECT | Evidence that additional participation improves value for existing users | Multi-homing, low interaction, declining engagement, subsidized participation |
| COST_ADVANTAGE | Sustained comparable unit-cost advantage with a structural mechanism | Temporary input windfall, subsidy dependence, peer convergence, diseconomies |
| EFFICIENT_SCALE | Limited market demand and evidenced entry economics that discourage duplication | New entrants, cheaper technology, overcapacity or market boundaries changing |

These examples are our proposed test questions, not Morningstar rating criteria.
Distribution and ecosystem advantages may explain mechanisms but are not automatic
sixth/seventh moat scores. A high margin or recent winner is not proof of a moat.

Future evolution concepts: `MOAT_STRENGTHENING`, `MOAT_STABLE`, `MOAT_WEAKENING`,
`UNKNOWN`. They describe changes in a scoped hypothesis, not a production rating.
STABLE requires comparable evidence across time; no new evidence means UNKNOWN or
stale, not stability. Store staleness/coverage separately from the direction label.
Conflicting evidence yields a disputed assessment with retained alternatives.

A future rubric must specify required fields, evidence eligibility, comparability
and decision rationale. Validation and version selection can be deterministic;
interpretation may require recorded human review. LLMs may draft candidate assertions
with citations but cannot assign approved labels on their own. No numeric scoring,
weights, wide/narrow rating clone or production assignment is authorized now.

## CompanyStrategicEvents: what is happening now?

This is a company-focused extension of the [canonical event plan](NEWS_AND_EVENT_INTELLIGENCE_PLAN.md),
not a competing news store. One occurrence may link several companies/roles and
multiple source assertions. Preserve disagreements about that occurrence.

Proposed event types: NEW_PRODUCT, PRODUCT_GENERATION, CUSTOMER_WIN, DESIGN_WIN,
CAPACITY_EXPANSION, NEW_FACTORY, NEW_GEOGRAPHY, NEW_PARTNERSHIP, NEW_PRICING,
NEW_DISTRIBUTION, ACQUISITION, DIVESTITURE, RD_EXPANSION, MANAGEMENT_CHANGE,
RESTRUCTURING, DEBT_ISSUANCE, SHARE_REPURCHASE, CUSTOMER_LOSS, SUPPLIER_PROBLEM,
COMPETITIVE_LAUNCH. Version the taxonomy; retain provider-native types.

Required fields: event/version ID, company and counterpart roles, event_type,
event_date/effective interval, published_at, available_at, retrieved_at, timestamp
precision/timezone, source/version, evidence locators, estimated mechanism,
confidence/reasons, assertion type, status and correction links. Mechanism is an
attributed interpretation, not an observed revenue contribution. Preserve direction
by affected dimension rather than forcing every event into positive/negative.

Lifecycle: PROPOSED / ANNOUNCED / CONFIRMED / IN_PROGRESS / COMPLETED / DELAYED /
CANCELLED / UNKNOWN, with evidence for each transition. A design win is not booked
revenue; a factory announcement is not operational capacity; repurchase authorization
is not shares bought. Missing event_date can coexist with known publication time.
Avoid fabricated exact dates for ranges or anticipated events.

An event links to thesis catalysts, risks and expected evidence. Later completion
records do not change its earlier ANNOUNCED state. Event detection must work on
historical versions and survive duplicate syndication, ticker reuse and delayed receipts.

## Professional research contract and use policy

Provider-neutral `ProfessionalResearchArtifact` design:

| Group | Fields / required semantics |
| --- | --- |
| Identity | Provider/profile, report ID and version, company/entity mappings, title, analyst identity if available AND licensed; null with reason otherwise |
| Timing | published_at, available_at, retrieved_at, precision/timezone, historical-version certification and correction/supersession links |
| Attributed research | Business thesis, bull factors, bear factors, valuation assumptions, risk factors, moat evidence and capital-allocation observations; each linked to a locator and marked external opinion/forecast where applicable |
| Provenance | Raw object/hash or permitted reference-only locator, original format, extraction/parser/prompt version, author/reviewer and coverage limitations |
| Rights | Licensing classification, entitlement reference, allowed users/purposes, retention/expiry, machine extraction, AI processing, training, derived use and redistribution permissions; unknown is explicit |
| Interpretation | Original wording/scale when retention is permitted; any internal normalization is a separate versioned mapping with rationale, never an invented analyst forecast |

No report field is mandatory if the source does not supply it; absent differs from
neutral. Keep outside estimates separate from the canonical contributor-event feed:
a broker PDF or opinion is not automatically an eligible consensus member/revision.

| Access mode | Default architectural treatment |
| --- | --- |
| HUMAN_READABLE_REFERENCE_ONLY | Authorized human reference; metadata/citation only where permitted; no automatic bulk extraction or retained text |
| USER_SUPPLIED_DOCUMENT | Owner-supplied legitimate document for their own research; scoped analysis, access control and retention per rights; upload grants no redistribution or training rights |
| LICENSED_STRUCTURED_FEED | Approved contract controls ingestion fields, historical versions, users, derivative use and retention; adapter qualification still required |
| REDISTRIBUTABLE_DATA | Explicit license permits specified sharing; attribution and other conditions still apply; not synonymous with unrestricted use |

Morningstar methodology may be studied as an intellectual framework. Subscriber
reports may be reviewed when legitimately supplied by the project owner for their
own research, within applicable permissions. The architecture must not depend on
unauthorized automated scraping or redistribution. Systematic Morningstar data
requires verified licensing/API rights first. No scraping is implemented or proposed
as a workaround. This is a project use policy, not a determination of any user's license.

Before feeding text to external AI services, verify processing permissions and
retention terms. Restrict access to raw text and derived summaries as required.
On entitlement expiry, prevent further access and follow contract deletion rules;
retain permitted audit metadata/tombstones. Do not promise unrestricted perpetual
raw retention. Public Git contains only synthetic/permitted examples and contracts.

Historical replay requires the exact report version available at T, not today's
company page, latest fair value, or a retrospective analyst note. Separate document
publication from receipt and from extraction time. Certified historical archives
may support reconstructed runs; unverified documents enter only at receipt. Freeze
the extracted representation so a later LLM/model cannot rewrite past interpretations.

## Disagreement and corroboration

A `ResearchDisagreement` records subject/claim scope, target metric/basis/horizon,
research T, each view's assertion/version, family, direction, evidence, confidence,
and mismatch reason. Keep internal research, professional analysis, sell-side
consensus, market behavior and management separate. UNKNOWN and NOT_COMPARABLE
are valid outcomes. A bullish 24m business view and weak 21-day market trend may
be a horizon difference rather than a factual contradiction.

Synthetic example at T: internal thesis positive; professional view negative
because of valuation; same-target estimate revisions positive; market trend negative.
Display all four, their horizons and evidence. Do not average their signs or force
agreement. Record subsequent outcomes for the precise claims each made.

`ClaimCorroboration` links a management claim to supporting, contradicting and
non-informative evidence. Store claim ID/version, scope, as-of timestamp, evidence
IDs, independence/common-origin groups, confidence dimensions, rationale, rubric
version, reviewer and state: SUPPORTED / CONTRADICTED / MIXED / UNRESOLVED.

Example: management says demand is accelerating. Analyst revisions, job postings,
supplier activity, customer announcements, CapEx, pricing, web traffic and market
volume are possible checks. Revisions may echo management; hiring may replace
attrition; supplier output may serve other customers; volume indicates trading
attention, not demand. Only matched scope/time and a plausible mechanism justify
support. Trace common origin to prevent circular corroboration. Independent evidence
can strengthen confidence without proving causality or future revenue.

## Thesis graph and later learning

Use typed nodes: CLAIM, CAUSE, EXPECTED_OUTCOME, CATALYST, RISK,
INVALIDATION_CONDITION, EVIDENCE. Edges include SUPPORTED_BY, CONTRADICTED_BY,
HYPOTHESIZED_CAUSE, EXPECTS, DEPENDS_ON and INVALIDATED_BY. A causal edge remains a
hypothesis unless separately established; graph structure cannot manufacture causality.

Synthetic example: claim “acceleration should continue”; proposed cause “new
hyperscaler adoption”; catalyst “production ramp”; expected outcomes “revenue growth
> X, margin > Y and positive same-target revisions”; risks “qualification delay or
customer loss”; invalidation “delay beyond deadline or documented loss.” X/Y,
absolute fiscal targets, baseline, units and deadlines must be fixed before
publication, not left as vague words or tuned after outcomes.

Each node has stable ID, immutable version, company/scope, author/attribution,
available_at, research T, evidence links and confidence. Evaluatable nodes add
metric/basis, threshold, horizon, evidence policy and adjudication version. Freeze
nodes AND edges in the prediction packet. A revised thesis is a new packet linked
to the prior one; it cannot remove an old risk or change its deadline.

Evaluate each node separately as pending/supported/contradicted/mixed/indeterminate,
with risk occurrence, catalyst completion and invalidation recorded independently
of stock returns. See [ledger extension](RESEARCH_PREDICTION_AND_OUTCOME_LEDGER.md#qualitative-claims-disagreements-and-evidence-family-outcomes).
Ask which family was informative on matched claims and cohorts, not which source
“won” a narrative debate. Test marginal value with out-of-sample ablations and
coverage controls; shared sources and selection bias prevent naive causal attribution.
Thousands of recorded judgments can support learning only under the existing
chronological, maturity, multiple-testing and champion/challenger gates.

## Business quality to valuation, without displacing momentum

```text
business quality + competitive advantage duration + reinvestment opportunity
+ ROIC + growth + margin + cash flow + risk
→ explicit assumptions and bear/base/bull scenarios → DCF / intrinsic valuation
```

This is a proposed modeling bridge, not a computed formula. Link each scenario
parameter to evidence or mark it an analyst assumption. Competitive duration informs
profit persistence; capital intensity/reinvestment constrain growth and cash flows;
concentration/financing risks widen scenario uncertainty. ROIC needs a separately
versioned definition and comparable capital base. Do not infer it from margins alone.
Show sensitivities, applicability and missing inputs; avoid counting the same risk
in several adjustments without explanation. Estimated intrinsic value is not an
observed fact or promise of near-term price convergence.

Preserve fundamentals, market, estimates, valuation, moat/business quality, events,
professional research and alternative evidence separately. **Do not collapse them
into one unexplained AI score.** Later validated models may combine families only
with inspectable contributions, interactions, missingness and versions. AI can
summarize the packet, not override deterministic values or approved evidence states.

## Gates and sequencing

Existing PIT/data-source remediation comes first. These concepts extend the already
planned 2.8 snapshot/claim contract, 2.9 outcome evaluation, 2.10 learning controls,
2.11 deeper research/AI, and proposed 2.12 events/2.13 expanded sources; no renumbering
or phase start. Alternative-data access is unqualified until source-specific rights,
coverage and timestamp evidence pass review. See [alternative roadmap](ALTERNATIVE_DATA_ROADMAP.md).

Future acceptance: exact-source trace; management opinion distinct from observation;
same-release syndication not independent; old snapshot unchanged after corrected
report; minority contradiction preserved; no unsupported moat label; outcome unknown
when coverage is absent; same target/horizon comparison; restricted document not
exported; LLM summary cannot replace data. Use adversarial synthetic fixtures and
independently reviewed held-out documents before any scoring or production use.
