# Phase 2.2 Handoff Document

## 1. PROJECT
- **Repo:** `~/Document/investment-research-system`
- **Branch:** `master`
- **Last Clean Checkpoint:** `14720d4`
- **Environment:** PostgreSQL / Docker
- **Current Phase:** 2.2 Fundamental Feature Store
- **Note:** Phase 2.1 is COMPLETE and must not be redesigned.

## 2. PHASE 2.2 CURRENT STATE
- **Status:** NOT ACCEPTED.
- **Latest Forensic Audit Verdict:** `PHASE 2.2: FAIL`
- **Working Tree:** Contains uncommitted Phase 2.2 work.

## 3. CURRENT GIT STATE
- **Modified Files:**
  - `src/investment_research/cli/main.py`
  - `src/investment_research/models/research.py`
  - `src/investment_research/normalization/sec.py`
- **Untracked Files:**
  - `src/investment_research/research/`
  - `tests/unit/test_fundamental_features.py`

## 4. MAJOR DISCOVERED DEFECTS

### A. Fiscal Period Identity Error
- **Issue:** Resolver derives quarter from calendar month: `(month - 1) // 3 + 1`.
- **Failure:** Invalid for non-calendar fiscal years. Example: NVDA (fiscal year ends Jan). April 26, 2026 is fiscal Q1, but logic labels it Q2.
- **Rule:** Duration alone must NEVER establish fiscal quarter identity.

### B. Resolution Logic Failure
- **Issue:** Quarterly vs YTD resolution is broken.
- **Failure:** Correctly classified 90-day quarterly facts are still failing resolution in adversarial cases.

### C. Revenue QoQ Failure
- **Issue:** NVDA output has no valid sequential quarterly Revenue QoQ observations.
- **Rule:** Do NOT solve this by weakening period matching.

### D. Cost of Revenue Mapping Gap
- **Issue:** `CANONICAL_COST_OF_REVENUE` $\rightarrow$ `[]` $\rightarrow$ 0 matches for NVDA.
- **Action:** Investigate actual SEC tags and normalization architecture. Do not fabricate mappings.

### E. FCF Missing
- **Issue:** NVDA FCF observations are absent.
- **Root Cause:** CapEx is missing or resolver cannot correctly pair CFO and CapEx.

### F. EPS Value Mismatch
- **Issue:** Stored result differs from manual calculation.
- **Example:** Current 4.85 vs prior 3.14 $\approx$ 54% YoY, but stored result was 1.63.

### G. Cross-Company Data Gap
- **Issue:** Only NVDA has data. AVGO, CRDO, ANET, MU = 0.
- **Action:** Determine if cause is ingestion/universe/linkage/normalization/persistence.

### H. Foundational Data Integrity Risk
- **Issue:** `normalization/sec.py` was modified to add `period_start` extraction.
- **Impact:** NVDA `FinancialFact` count increased from $\approx$1,100 to 1,465.
- **Risk:** `on_conflict_do_nothing` created duplicates where some records have `period_start NULL` and others have it populated.
- **Rule:** Do not silently rewrite foundational SEC data.

### I. Test Insensitivity
- **Issue:** Mutation testing showed `available_at` propagation test passed even when `max()` was removed.
- **Action:** Repair adversarial and mutation tests.

## 5. CURRENT VERIFIED GOOD AREAS
- PIT amendment synthetic scenarios work.
- `available_at` max propagation implementation exists.
- 50-row provenance sample passed.
- Idempotency real run verified (no duplicates on repeated runs).
- `calculation_version` coexistence works.
- Security audit passed.
- Some Revenue YoY, Gross Margin, and Operating Margin calculations are correct.

## 6. CURRENT FEATURE REGISTRY
1. `revenue_yoy_growth`
2. `revenue_qoq_growth`
3. `revenue_growth_acceleration`
4. `gross_margin`
5. `gross_margin_yoy_change`
6. `operating_margin`
7. `operating_margin_yoy_change`
8. `operating_income_yoy_growth`
9. `eps_yoy_growth`
10. `eps_growth_acceleration`
11. `operating_cash_flow_yoy_growth`
12. `free_cash_flow`
13. `fcf_yoy_growth`
14. `fcf_margin`
15. `fcf_conversion`

## 7. CORE ARCHITECTURAL RULES

### Temporal Triad
- `period_end`: Economic/reference date.
- `available_at`: When information became knowable.
- `calculated_at`: Calculation timestamp.

### Backtest Eligibility
- `available_at <= T`

### FeatureValue PIT
- `available_at` must equal the maximum `available_at` of ALL source inputs.

### Period Matching Hierarchy
1. Same fiscal period identity
2. Same fiscal-year relationship
3. Compatible duration/period structure
4. Bounded date tolerance (only if justified)

**NEVER:**
- Use date proximity alone.
- Use calendar quarter alone.
- Use duration alone.
- Allow $\pm 14$ days to override fiscal semantics.
- Weaken matching rules to increase coverage.

### Historical Universe
- `start_date` inclusive, `end_date` exclusive.

### Version Semantics
- `research_features.version`: Feature definition/spec version.
- `feature_values.calculation_version`: Implementation/calculation version.
- Values are append-only/versioned.

## 8. IMPORTANT NEXT-SESSION OBJECTIVE
**Do NOT start coding immediately.**
1. Inspect actual repository/database $\rightarrow$ produce compact forensic diagnosis.
2. Fix architecture in small controlled steps:
   - STEP 1: Understand SEC `FinancialFact` schema and normalization.
   - STEP 2: Fix fiscal-period identity correctly.
   - STEP 3: Fix comparable and sequential period resolution.
   - STEP 4: Fix Revenue QoQ/YoY/acceleration.
   - STEP 5: Fix Cost of Revenue mapping (if supported).
   - STEP 6: Fix EPS resolution/calculation.
   - STEP 7: Fix CFO/CapEx/FCF.
   - STEP 8: Investigate missing data for AVGO, CRDO, ANET, MU.
   - STEP 9: Repair adversarial and mutation tests.
   - STEP 10: Run full evidence audit.

**No commit or push until explicitly instructed.**

## 9. REQUIRED VALIDATION COMPANIES
- NVDA, AVGO, CRDO, ANET, MU

## 10. MOST IMPORTANT PRINCIPLE
**Correctness > Coverage.**
- Do not use fuzzy matching to fill gaps.
- Preserve missing states and explain why.
- Do not fabricate fiscal metadata.
- Do not silently rewrite foundational SEC facts.
