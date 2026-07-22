---
name: comp-sheet
description: Build an industry comp sheet Excel model with deep operational KPIs
argument-hint: TICKER
---

Build a multi-company industry comp sheet Excel model for the company specified by the user: $ARGUMENTS

This produces an interactive `.xlsx` workbook — the kind of comp sheet every analyst on a coverage team maintains. Multi-company, multi-tab, with deep operational KPIs alongside standard financials.

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow the data access detection logic and design system throughout this skill.

Follow these steps:

## 1. Company & Peer Setup

Look up the target company by ticker using `discover_companies`. Capture `company_id`, `latest_calendar_quarter` (anchor for all period calculations — see `data-access.md` Section 1.5), and `latest_fiscal_quarter`. Note the firm name for report attribution (default: "Daloopa") — see `data-access.md` Section 4.5.

Then identify 6-10 comparable companies using the same logic as `/comps`:
- **Direct competitors** in the same market
- **Business model peers** (similar revenue model)
- **Size peers** (similar market cap range)
- **Growth profile peers** (similar growth rate)

Look up all peer company_ids via Daloopa. If a peer isn't available in Daloopa, include it with market data only and note the limitation.

List the full peer group with brief justification for each.

## 2. Deep Data Gathering

For each company (target + all peers), pull from Daloopa:

**Calculate 8 quarters backward from `latest_calendar_quarter`. Pull financials:**
- Revenue, Gross Profit, Operating Income, Net Income, Diluted EPS
- Operating Cash Flow, Capital Expenditures, D&A
- Free Cash Flow (compute as OCF - CapEx)
- R&D Expense, SG&A (where available)

**Segment revenue breakdown** (all available segments, 8 quarters)

**Company-specific operational KPIs** — use the 9-sector taxonomy to know what to search for:
- **SaaS/Cloud**: ARR, net revenue retention, RPO/cRPO, customers >$100K, cloud gross margin
- **Consumer Tech**: DAU/MAU, ARPU, engagement metrics, installed base, paid subscribers
- **E-commerce/Marketplace**: GMV, take rate, active buyers/sellers, order frequency
- **Retail**: same-store sales, store count, average ticket, transactions
- **Telecom/Media**: subscribers, churn, ARPU, content spend
- **Hardware**: units shipped, ASP, attach rate, installed base
- **Financial Services**: AUM, NIM, loan growth, credit quality metrics, fee income ratio
- **Pharma/Biotech**: pipeline stage, patient starts, scripts, market share
- **Industrials/Energy**: backlog, book-to-bill, utilization, production volumes, reserves

**Stock prices & valuation multiples:**
Use `get_stock_prices` (see `data-access.md` Section 1.7) to pull prices for ALL companies in a single batch call. Get:
- Current price: `dates` = 3 most recent calendar days for all company_ids
- Quarter-end prices: `dates` = quarter-end dates matching the financial periods (for historical multiples)

Then compute valuation metrics by combining stock prices with Daloopa fundamentals:
- **Market Cap** = Close price × Diluted shares outstanding
- **Enterprise Value** = Market Cap + Total Debt - Cash
- **P/E (trailing)** = Market Cap / Net Income (trailing 4Q)
- **EV/EBITDA** = EV / EBITDA (trailing 4Q)
- **P/S** = Market Cap / Revenue (trailing 4Q)
- **P/B** = Market Cap / Total Equity
- **EV/FCF** = EV / Free Cash Flow (trailing 4Q)
- **FCF Yield** = FCF (trailing 4Q) / Market Cap
- **Dividend Yield** = Dividends Paid (trailing 4Q) / Market Cap

For beta, use MCP market data tools if available, otherwise web search (see `data-access.md` Section 2). For forward multiples, use consensus estimates if available (Section 3).

## 3. KPI Discovery & Mapping

After pulling data, build the KPI mapping:
- Which KPIs are available for which companies? Build a coverage matrix.
- Group KPIs into categories:
  - **Segment Revenue**: product/service line breakdowns
  - **Growth KPIs**: subscriber growth, unit growth, same-store sales growth
  - **Unit Economics**: ARPU, ASP, take rate, retention
  - **Efficiency**: R&D % of revenue, SBC % of revenue, CapEx % of revenue
  - **Engagement**: DAU/MAU, retention, churn
- Flag KPIs that are comparable across peers vs company-specific

## 4. Compute Derived Metrics

For each company, calculate:

**Margins:**
- Gross Margin, Operating Margin, Net Margin, FCF Margin (each quarter)

**Growth rates:**
- Revenue YoY, EPS YoY, segment revenue YoY (each quarter where year-ago data exists)

**Capital metrics:**
- Net Debt (Total Debt - Cash)
- Net Debt/EBITDA
- Shareholder Yield (Buybacks + Dividends) / Market Cap

**Historical multiples (from quarter-end prices pulled in Section 2):**
- Compute P/E, EV/EBITDA, P/S, EV/FCF at each quarter-end to show how multiples have trended
- This lets the reader see whether the current multiple is elevated or depressed vs. the company's own history

**Implied valuation:**
- For each valuation methodology (P/E, EV/EBITDA, P/S, EV/FCF):
  - Peer median multiple × target metric = implied value
  - Convert to implied share price
- Compute median implied price across methodologies

## 5. Structure the Data

Organize the gathered data as a multi-company dataset in memory, ready to feed the workbook builder in the next step:

```json
{
  "target_ticker": "AAPL",
  "as_of_date": "YYYY-MM-DD",
  "companies": [
    {
      "ticker": "AAPL",
      "name": "Apple Inc.",
      "is_target": true,
      "market_data": {
        "price": ..., "market_cap": ..., "enterprise_value": ...,
        "shares_outstanding": ..., "beta": ...,
        "trailing_pe": ..., "forward_pe": ...,
        "ev_ebitda": ..., "price_to_sales": ...,
        "ev_fcf": ..., "dividend_yield": ...
      },
      "periods": ["2024Q1", "2024Q2", ...],
      "financials": {
        "Revenue": {"2024Q1": ..., ...},
        "Gross Profit": {...}, ...
      },
      "margins": {
        "Gross Margin": {"2024Q1": ..., ...}, ...
      },
      "growth": {
        "Revenue Growth YoY": {"2024Q1": ..., ...}, ...
      },
      "kpis": {
        "iPhone Revenue": {"2024Q1": ..., ...}, ...
      },
      "kpi_categories": {
        "Segment Revenue": ["iPhone Revenue", "Services Revenue", ...],
        "Growth KPIs": ["Services Growth YoY"],
        "Efficiency": ["R&D % Revenue", "SBC % Revenue"]
      }
    },
    ...more companies...
  ],
  "implied_valuation": {
    "pe_implied": ...,
    "ev_ebitda_implied": ...,
    "ps_implied": ...,
    "ev_fcf_implied": ...,
    "median_implied": ...
  }
}
```

Every datapoint that originates from Daloopa must carry its `fundamental_id` alongside the value so the hyperlink can be attached when the workbook is built.

## 6. Build the Workbook

Generate a React artifact that uses the SheetJS (`xlsx`) library to build and download the `.xlsx` workbook client-side. The artifact should construct the workbook from the structured data above and trigger a download when the user clicks a button (or immediately on load).

!!! MANDATORY EXCEL DATAPOINT HYPERLINK FORMAT !!!
- ALWAYS attach a cell hyperlink when writing Daloopa fundamental datapoints into a worksheet cell
- Format: [$<value> <unit>](https://daloopa.com/src/{fundamental_id})
- Example: write `123.4` to cell B5 and set its link (`cell.l = { Target: "https://daloopa.com/src/71667434" }` in SheetJS) so it points to the source
- Every Daloopa-sourced numeric cell MUST include its clickable source link — do not put the URL in a neighboring cell or as plain text
- Do NOT add datapoint hyperlinks to headers, labels, blank cells, or unsourced formulas

The workbook must have 8 tabs (sheets), built in this order:
1. **Comp Summary** — one-pager with all companies, multiples, implied valuation
2. **Revenue Drivers** — unit economics decomposition per company (trailing 4Q)
3. **Operating KPIs** — cross-company KPI comparison matrix
4. **Financial Summary** — side-by-side income statements (trailing 4Q)
5. **Growth & Margins** — trend analysis (up to 8Q)
6. **Valuation Detail** — implied prices by methodology, premium/discount
7. **Balance Sheet & Capital** — leverage and capital returns
8. **Raw Data** — full quarterly appendix for each company

Use SheetJS formulas (e.g. `cell.f`) where the original design calls for computed values (margins, growth rates, implied valuation) so the workbook stays live and auditable, rather than hardcoding pre-computed numbers.

## 7. Output

Confirm to the user that the `.xlsx` workbook has downloaded from the artifact.

Highlight in your summary:
- **Target positioning vs peers**: Where does it rank on growth, margins, and valuation?
- **Most differentiated KPIs**: Which operational metrics set the target apart (positive or negative)?
- **Implied valuation range**: What does the peer group suggest the stock is worth?
- **Key risk**: What's the biggest vulnerability the comp sheet reveals (e.g., premium valuation with decelerating KPIs, margins below peers, etc.)?

All financial figures in the summary must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
