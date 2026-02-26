# Data Access Reference

All skills that need financial data should follow this reference. Read `design-system.md` (co-located with this file) for formatting, analytical density, and styling conventions.

---

## Section 1: Daloopa MCP Tools

Check your available tools. If you see Daloopa MCP tools (`discover_companies`, `discover_company_series`, `get_company_fundamentals`, `search_documents`), MCP is available.

| Operation | MCP Tool |
|---|---|
| Find company by ticker/name | `discover_companies(keywords=["TICKER"])` |
| Find available series/metrics | `discover_company_series(company_id, keywords, periods)` |
| Pull financial data | `get_company_fundamentals(company_id, periods, series_ids)` |
| Search SEC filings | `search_documents(keywords, company_ids, periods)` |

Results come back as structured data you can use directly.

If MCP is not available, tell the user to verify their Daloopa MCP connection by running `/daloopa:setup`.

## Section 2: External Market Data

Skills that need market-side data should gather the following. Use whatever tools or data sources are available in your environment.

| Data Need | What to Get |
|---|---|
| **Stock quote** | Current price, market cap, shares outstanding, beta |
| **Trading multiples** | Trailing P/E, Forward P/E, EV/EBITDA, P/S, P/B, dividend yield |
| **Historical prices** | OHLCV data for trend analysis (1-5 years) |
| **Peer multiples** | Side-by-side trading multiples for 5-10 comparable companies |
| **Risk-free rate** | 10Y Treasury yield (for WACC/DCF calculations) |

If market data is unavailable, note the limitation and proceed with Daloopa fundamentals only. Use reasonable defaults where needed (beta=1.0, risk-free rate=4.5%).

## Section 3: Consensus Estimates (Optional)

When available, consensus analyst estimates add valuable context. Look for:

| Data Need | Use Case |
|---|---|
| **Consensus revenue / EPS** | Beat/miss analysis vs. Street expectations |
| **Forward estimates (NTM)** | Forward P/E, forward EV/EBITDA for comps |
| **Estimate revisions** | Trend in analyst expectations (up/down/stable) |
| **Price targets** | Consensus target and range for context |

If consensus data is not available, skip these sections and note "consensus data not available" rather than guessing.

## Section 4: Citation Requirements (MANDATORY)

**Every financial figure sourced from Daloopa MUST include a citation link.** This is non-negotiable.

Format: `<a href="https://daloopa.com/src/{fundamental_id}">$X.XX million</a>`

The `fundamental_id` (or `id`) is returned in every `get_company_fundamentals` response. You must:

1. **Capture the `fundamental_id` at data-pull time** — when you call `get_company_fundamentals`, record the `id` for every value
2. **Carry the ID through to output** — when building tables, prose, or context JSON, attach the citation link to every Daloopa-sourced number
3. **Never drop citation IDs** — if a value came from Daloopa, it gets a link. No exceptions. Computed values (e.g., margins, growth rates) derived from Daloopa figures should cite the underlying inputs
4. **Document citations** — when quoting SEC filings from `search_documents`, link to: `[Document Name](https://marketplace.daloopa.com/document/{document_id})`

If you output a financial figure without a citation, it cannot be verified. Uncitable numbers are useless to an analyst.
