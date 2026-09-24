# Data sources

Use [Data-source trust and PIT](DATA_SOURCE_TRUST_AND_PIT.md) for temporal contracts
and [Provider risk register](DATA_PROVIDER_RISK_REGISTER.md) for coverage, rights,
revision and replacement risks. A historical endpoint is not proof of PIT data.

| Source | Current repository use | Limitation |
| --- | --- | --- |
| SEC Company Facts / EDGAR | Company/fundamental ingestion and filing evidence foundation | Date-only fundamental visibility and comparative contexts need qualification |
| Yahoo chart | Direct market connector, OHLCV normalization | Not yfinance; raw price/action semantics unqualified for historical PIT |
| FRED | API-key macro ingestion | Current history, not a vintage-preserving replay system |
| RSS | Basic news/article ingestion | No production structured event intelligence or certified historical archive |
| Synthetic estimates | Full estimate pipeline qualification tests | No licensed production analyst-event provider |

ALFRED, institutional market/estimate providers and additional research sources
remain future work. Never infer storage, redistribution or AI-processing rights
from endpoint accessibility or a connector's software license.
