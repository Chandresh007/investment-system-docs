# Data Sources

## SEC EDGAR
- **Role**: Primary source for fundamental financial data.
- **Endpoints**: Company Tickers, Company Facts, Submissions.
- **Auth**: No key required, but must use a compliant User-Agent.
- **Rate Limit**: 10 requests per second.

## FRED (Federal Reserve Economic Data)
- **Role**: Macroeconomic indicators.
- **Auth**: API Key required (`FRED_API_KEY`).
- **Endpoints**: Series metadata, Observations.
- **Key Series**: FEDFUNDS, CPIAUCSL, GDP, etc.
- **Rate Limit**: Standard FRED API limits.

## Market Data
- **Role**: Historical price and volume data.
- **Strategy**: Abstracted via `MarketDataConnector` to allow provider swapping.

## News
- **Role**: Event-driven signal detection.
- **Strategy**: Connector interface defined; implementation planned for later Phase 1 stages.
