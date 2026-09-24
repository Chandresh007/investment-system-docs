# Foundation and Phase 2.5 freeze review

Review date: 2026-09-22. Starting commit:
`fdd8b7a778547677aaa4e388480744639c91bc89` on `master`, equal to
`origin/master` with a clean tree. The fixes below are published by the commit
named `Freeze Phase 2.5 and qualified foundation`.

## Decision

**PASS — FREEZE APPROVED**

- Foundation: **FROZEN for implemented semantics**.
- Phase 2.5: **FROZEN**.
- Phase 2.6: **READY TO START**.

This freezes architecture, formulas, temporal behavior and publication contracts.
It does not qualify unlicensed or untested live providers.

## Scope and findings

The review independently inspected the foundation models, migrations, resolvers,
provider policy, raw archive, SEC/macro/market paths, Phase 2.5 ingestion/replay/
consensus/revision/runner code, tests, synthetic lifecycle and Git history. Earlier
PASS reports were treated as claims to falsify.

| Area | Freeze evidence |
| --- | --- |
| Historical universe | Effective membership and knowledge time are separate; future announcements, late evidence, removals and re-entry cannot leak backward. |
| Market/actions | Yahoo is blocked from historical/survivorship claims; `close` and `adjusted_close` remain separate; only PIT-known actions enter normalization. |
| Macro | Ordinary FRED remains current-only; vintage identity preserves initial and revised values with distinct availability. |
| SEC | Acceptance time, conservative date-only fallback, amendments, comparative facts and restatements select the latest fact known by T without rewriting earlier knowledge. |
| Raw/provenance | New bytes are SHA-256 addressed, atomically non-overwriting and verified; original, reconstructed, missing, mismatch and legacy states remain distinct. |
| Feature history | Exact same identity/content is idempotent; changed data/build/version creates another artifact. PostgreSQL rejects UPDATE and DELETE. |
| Survivorship | Permanent security IDs and bitemporal identity survive ticker/lifecycle changes. No live profile has the complete capability set, so the runtime refuses the claim. |

## Phase 2.5 invariants

Direct code and test review confirmed absolute fiscal-target identity, one-time
horizon resolution at T, same-target historical lookbacks, stable contributor and
event identities, PIT correction replay, no fallback from the newest unusable
event/state, minimum-three consensus and 38-digit Decimal math.

The frozen formulas are:

```text
EPS scaled: 0 if old == 0 and new == 0,
            otherwise (new-old)/max(abs(old),abs(new))
Revenue:    (new-old)/old for valid positive old revenue
Breadth:    (U-D)/(U+D), one latest directional vote per contributor
Count:      value-changing economic events only
```

NEW, WITHDRAWAL, CORRECTION, duplicates and reaffirmations do not enter revision
count. Corrections change the represented historical event only after correction
availability and never become analyst revision activity. The registry contains
exactly 15 estimate features and remains isolated from fundamental/market runners.

## Defects reproduced and fixed

1. Runtime provider admission accepted catalog-unknown qualifications and failed
   to bind qualification version, an empty capability set, or the synthetic flag.
   Exact catalog contract validation now fails closed.
2. `EstimateFeatureRunner` could publish from an unqualified profile without a
   quality record. It now requires profile-matched catalog admission covering the
   full requested lookback and persists that admission. Explicit overrides force
   `DEVELOPMENT` and record the limitation.
3. Historical FeatureValue rows rejected UPDATE but allowed DELETE. Migration
   `fnd006` seals both operations.
4. `alembic check` found three model/schema index differences. `fnd006` adds the
   corporate-action availability and historical-universe sector/industry indexes.

Each defect had a failing regression before its fix. No financial formula changed.

## Synthetic path and provenance

The full Helios fixture traverses provider response/raw storage, normalization,
ingestion, fiscal/contributor resolution, correction-aware replay, consensus,
revision calculation, the feature runner and persisted FeatureValue rows. Final
states are calculated by production components rather than injected.

The persisted T3 scaled-EPS walk resolves FeatureValue → registry definition →
provider admission/profile → dataset/build → absolute Q2 target → current/prior
consensus → members/exclusions → observations → raw bytes → contributor aliases →
fiscal/reporting/coverage evidence. Raw bytes are hash-verified. Evidence IDs are
stored in FeatureValue JSON rather than a separate edge table; the referenced
estimate rows are immutable and the walk has no missing logical link.

## Migration and tests

- Full regression: **431 passed, 8 existing warnings, zero failures**.
- Focused estimates: **296 passed**.
- Focused foundation: **52 passed**.
- Alembic current/head: **fnd006**; `alembic check`: no upgrade operations.
- Disposable PostgreSQL: fresh base→head, `fnd005`→head, pre-estimate prior head→head,
  metadata comparison, and all focused tests passed.
- Disposable database/raw backup, isolated restore, hashes/query proof and tamper
  rejection passed. The persistent database was never downgraded.
- Legacy fundamental/foundation tests now use rollback-isolated transactions;
  their internal commits no longer leak rows or require destructive table cleanup.
- `git diff --check` passed. Phase 2.2 and 2.4 formula bodies remain unchanged;
  later changes are temporal selection, provenance and immutable publication.

## External limitations

The following remain independently blocked: institutional raw market/actions and
delisted-security coverage, a production analyst-estimate event provider, live
ALFRED qualification, broad accession-level Company Facts reconciliation and a
live off-host backup/restore drill. These are visible provider/deployment limits.
They do not weaken the frozen semantics, and none can obtain a stronger runtime
quality label without a new catalog qualification and evidence.
