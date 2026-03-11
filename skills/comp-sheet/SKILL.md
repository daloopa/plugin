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

**Market data** for each company (see data-access.md Section 2):
- Price, market cap, enterprise value, shares outstanding, beta
- All trading multiples: P/E (trailing + forward), EV/EBITDA, P/S, P/B, EV/FCF, dividend yield

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
- FCF Yield (trailing 4Q FCF / Market Cap)
- Shareholder Yield (Buybacks + Dividends) / Market Cap

**Implied valuation:**
- For each valuation methodology (P/E, EV/EBITDA, P/S, EV/FCF):
  - Peer median multiple × target metric = implied value
  - Convert to implied share price
- Compute median implied price across methodologies

## 5. Structure Data for Excel Export

Organize the data into 8 tabs (structure shown below). This will be rendered as a downloadable .xlsx using a React artifact with SheetJS:

**Tab 1: Comp Summary** — one-pager with all companies, multiples, implied valuation
- Company name, ticker, price, market cap, enterprise value
- Trading multiples (P/E, EV/EBITDA, P/S, P/B, EV/FCF, dividend yield)
- Implied valuation by methodology (P/E implied, EV/EBITDA implied, P/S implied, EV/FCF implied, median implied)
- Premium/discount to median

**Tab 2: Revenue Drivers** — unit economics decomposition per company (trailing 4Q)
- Segment revenue breakdown
- Key volume metrics (units, subscribers, stores, etc.)
- Key price metrics (ARPU, ASP, etc.)
- Revenue per unit, revenue per segment

**Tab 3: Operating KPIs** — cross-company KPI comparison matrix
- Rows = KPIs (grouped by category: Segment Revenue, Growth KPIs, Unit Economics, Efficiency, Engagement)
- Columns = companies
- Trailing 4Q averages or latest quarter values
- Sparse matrix (not all KPIs available for all companies)

**Tab 4: Financial Summary** — side-by-side income statements (trailing 4Q)
- Revenue, Gross Profit, Operating Income, Net Income, EPS
- R&D, SG&A, CapEx, D&A
- OCF, FCF
- All companies in columns, metrics in rows

**Tab 5: Growth & Margins** — trend analysis (up to 8Q)
- Revenue YoY growth, EPS YoY growth, segment growth
- Gross margin, operating margin, net margin, FCF margin
- Each metric with 8Q history per company

**Tab 6: Valuation Detail** — implied prices by methodology, premium/discount
- For each methodology (P/E, EV/EBITDA, P/S, EV/FCF):
  - Peer median multiple
  - Target metric (trailing 4Q or forward)
  - Implied equity value
  - Implied share price
- Median implied price across methodologies
- Current price vs implied (premium/discount %)

**Tab 7: Balance Sheet & Capital** — leverage and capital returns
- Total debt, cash, net debt
- Net debt/EBITDA
- Buybacks (trailing 4Q), dividends (trailing 4Q)
- FCF yield, shareholder yield

**Tab 8: Raw Data** — full quarterly appendix for each company
- All financials, margins, growth rates, KPIs by quarter (8Q)
- One section per company

## 6. Render Excel Artifact

Generate a React artifact that:
- Uses the SheetJS library (xlsx) to build a workbook with 8 tabs matching the structure above
- Applies basic styling (bold headers, number formatting, freeze panes)
- Downloads the .xlsx file to the user's browser with filename `{TICKERS}_comp_sheet.xlsx`

The artifact should include:
- Import statement for xlsx library (use CDN: https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js)
- Button to trigger download
- All 8 tabs pre-populated with the data from steps 1-4

## 7. Output

Tell the user that the `.xlsx` will download when they click the button in the artifact.

Highlight in your summary:
- **Target positioning vs peers**: Where does it rank on growth, margins, and valuation?
- **Most differentiated KPIs**: Which operational metrics set the target apart (positive or negative)?
- **Implied valuation range**: What does the peer group suggest the stock is worth?
- **Key risk**: What's the biggest vulnerability the comp sheet reveals (e.g., premium valuation with decelerating KPIs, margins below peers, etc.)?

All financial figures in the summary must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
