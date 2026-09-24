# SEC EDGAR and Company Facts qualification

- **Purpose:** authoritative filing evidence and normalized public fundamentals.
- **Profiles/versions:** `sec_edgar_filings` / `accession_acceptance_v1`,
  qualification `sec-edgar-2026-09-22-v1`; `sec_companyfacts` /
  `companyfacts_v1`, qualification `sec-companyfacts-2026-09-22-v1`.
- **Tested endpoints:** company submissions, Company Facts, and accession archive
  URLs. Tests use synthetic/mocked responses; no network test is required.
- **PIT semantics:** an original accession with archived bytes, accession identity,
  and SEC acceptance evidence can be `AUTHORITATIVE_PIT` for that filing scope.
  Company Facts is a current aggregate and remains `UNQUALIFIED` for historical
  replay until each selected value is reconciled to its filing and original bytes.
- **Revision behavior:** amendments and restatements are later accession versions.
  They remain separate facts and never backdate or replace earlier knowledge.
- **Coverage:** no complete issuer/tag/form/history coverage claim. Paper filings,
  custom taxonomies, tag changes, missing CapEx, and extraction changes remain gaps.
- **Delisted support:** EDGAR retains filer records but is not a qualified market,
  action, terminal-return, or survivorship provider.
- **Licensing/rights:** public access and fair-access rules apply; third-party
  exhibits and downstream redistribution/AI rights are not broadly certified.
- **Historical replay eligibility:** only the exact archived accession scope with
  pinned coverage and evidence. The current Company Facts adapter is rejected.
- **Production status:** `CONDITIONALLY_QUALIFIED_ACCESSION_SCOPE_ONLY` for original
  filings; `PIT_PARTIAL_ACCESSION_PROVENANCE_REQUIRED` for Company Facts.

## Availability rules

`acceptance_datetime` is parsed as New York time when the source omits an offset
and stored as an aware UTC instant. A filing accepted at 16:30 ET is unavailable
at 16:29:59 ET and eligible at 16:30 ET under this policy. SEC notes that website
availability often follows acceptance by one to three minutes, so strict operational
receipt research may choose the later receipt timestamp. When only a filing date
exists, the fact becomes eligible at the next New York midnight. Beginning-of-day
availability is forbidden.

Company Facts `fy`/`fp` identify the reporting filing context. They are preserved
again as `reporting_fiscal_year` / `reporting_fiscal_period`; a later comparative
record does not gain earlier knowledge time. The frozen Phase 2.2 fields remain for
compatibility, so production use still requires accession-level fiscal-context
reconciliation. The golden test covers a current quarter, a prior-year comparative
included later, an amendment, and a restatement-like changed value. As-of selection
returns the original before later filings and the later version only after its own
availability.

An earnings release before a 10-Q/10-K is separate evidence. The filing resolver
does not backdate a filing to an earlier press release. A future release adapter
must archive and qualify that source independently.
