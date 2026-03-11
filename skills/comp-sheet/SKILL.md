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
- Use the 3-step resolution order: (1) Check for MCP market data tools, (2) Web search if needed, (3) Use sensible defaults
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

## 5. Build Excel Workbook

Generate a React artifact that uses SheetJS (xlsx library) to build and download the .xlsx file directly in the user's browser.

The workbook must contain exactly 8 tabs with this structure:

### Tab 1: Comp Summary
One-pager with all companies side-by-side:
- Company name, ticker, price, market cap, enterprise value
- Trading multiples: P/E (trailing + forward), EV/EBITDA, P/S, P/B, EV/FCF, dividend yield
- Latest quarter financials: Revenue, Gross Margin, Operating Margin, Net Margin, FCF Margin
- Latest quarter growth: Revenue YoY, EPS YoY
- Implied valuation for target company (all methodologies + median)
- Rank target vs peers on each metric

### Tab 2: Revenue Drivers
Unit economics decomposition per company (trailing 4Q):
- Segment revenue breakdown ($ and % mix)
- Unit economics KPIs (ARPU, ASP, take rate, etc.)
- Per-segment growth rates
- Mix shift analysis

### Tab 3: Operating KPIs
Cross-company KPI comparison matrix:
- Rows = KPI categories (Segment Revenue, Growth KPIs, Unit Economics, Efficiency, Engagement)
- Columns = Companies (target first, then peers)
- Values = Latest quarter data
- Peer median and percentile ranking for target

### Tab 4: Financial Summary
Side-by-side income statements (trailing 4Q):
- Revenue, Gross Profit, Operating Income, Net Income, Diluted EPS
- R&D, SG&A, D&A, Operating Cash Flow, CapEx, Free Cash Flow
- All margins calculated
- Peer median column

### Tab 5: Growth & Margins
Trend analysis (up to 8Q):
- Revenue growth YoY by quarter
- Margin trends by quarter (Gross, Operating, Net, FCF)
- Sequential acceleration/deceleration
- Peer median trends

### Tab 6: Valuation Detail
Implied prices by methodology:
- P/E implied (trailing + forward)
- EV/EBITDA implied
- P/S implied
- EV/FCF implied
- Median implied price
- Current price vs implied (premium/discount %)
- Upside/downside to each methodology

### Tab 7: Balance Sheet & Capital
Leverage and capital returns:
- Total Debt, Cash, Net Debt
- Net Debt/EBITDA
- FCF Yield
- Shareholder Yield (buybacks + dividends)
- Shares outstanding change (YoY)
- Peer comparison

### Tab 8: Raw Data
Full quarterly appendix for each company:
- All 8 quarters of financial data
- All 8 quarters of KPI data
- All 8 quarters of derived metrics
- Citation footnotes with Daloopa fundamental IDs

**Styling requirements (follow design-system.md):**
- Header row: Navy (#1B2A4A) background, white text, bold
- Target company column: Gold (#C5A55A) background (light tint)
- Peer median column: Steel Blue (#4A6FA5) background (light tint)
- Growth cells: Green (#27AE60) for positive, Red (#C0392B) for negative
- Number formats: $X.Xbn for revenue/market cap, X.X% for margins/growth, X.Xx for multiples
- Freeze top row and leftmost column in all tabs
- Auto-fit column widths

The React artifact should:
1. Structure all data in JavaScript objects matching the 8-tab layout
2. Use SheetJS to create the workbook programmatically
3. Apply all styling (cell colors, number formats, font weights)
4. Provide a download button that saves as `{TARGET_TICKER}_comp_sheet.xlsx`

## 6. Output

Present the React artifact with the download button.

Below the artifact, provide a summary highlighting:
- **Target positioning vs peers**: Where does it rank on growth, margins, and valuation?
- **Most differentiated KPIs**: Which operational metrics set the target apart (positive or negative)?
- **Implied valuation range**: What does the peer group suggest the stock is worth?
- **Key risk**: What's the biggest vulnerability the comp sheet reveals (e.g., premium valuation with decelerating KPIs, margins below peers, etc.)?

All financial figures in the summary must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
