---
name: build-model
description: Build a multi-tab Excel financial model
argument-hint: TICKER
---

Build a comprehensive Excel financial model (.xlsx) for the company specified by the user: $ARGUMENTS

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow the data access detection logic and design system throughout this skill.

This skill gathers all available financial data and builds a multi-tab Excel model from scratch using a React artifact with SheetJS (xlsx library).

## Phase 1 — Company Setup
Look up the company by ticker using `discover_companies`. Capture:
- `company_id`
- `latest_calendar_quarter` — anchor for all period calculations (see `data-access.md` Section 1.5)
- `latest_fiscal_quarter`
- Firm name for report attribution (default: "Daloopa") — see `data-access.md` Section 4.5

Get current stock price, market cap, shares outstanding, beta, and trading multiples for {TICKER} (see data-access.md Section 2 for how to source market data).

## Phase 2 — Comprehensive Data Pull
Calculate periods backward from `latest_calendar_quarter`. Pull as much data as Daloopa has for this company. Target 8-16 quarters.

**Income Statement — search and pull all available:**
- Revenue / Net Sales
- Cost of Revenue / COGS
- Gross Profit
- Research & Development
- Selling, General & Administrative
- Total Operating Expenses
- Operating Income
- Interest Expense / Income
- Pre-tax Income
- Tax Expense
- Net Income
- Diluted EPS
- Diluted Shares Outstanding
- EBITDA (or compute from Op Income + D&A)
- D&A

**Balance Sheet — search and pull all available:**
- Cash and Equivalents
- Short-term Investments
- Accounts Receivable
- Inventory
- Total Current Assets
- PP&E (net)
- Goodwill
- Total Assets
- Accounts Payable
- Short-term Debt
- Long-term Debt
- Total Liabilities
- Total Equity

**Cash Flow — search and pull all available:**
- Operating Cash Flow
- Capital Expenditures
- Depreciation & Amortization
- Acquisitions
- Dividends Paid
- Share Repurchases
- Free Cash Flow (compute if not direct)

**Segments:**
- Revenue by segment
- Operating income by segment (if available)

**KPIs:**
- All company-specific operating metrics

**Guidance:**
- All guidance series and corresponding actuals

## Phase 3 — Market Data & Peers
- Identify 5-8 peers and get their trading multiples (see data-access.md Section 2)
- Get risk-free rate (see data-access.md Section 2)
- If consensus forward estimates are available (data-access.md Section 3), include NTM estimates for peers

## Phase 4 — Projections
Build forward estimates using the following methodology:
- Revenue: Start from latest guidance if available, decay to long-term growth rate (5-7%) over projection period
- Gross Margin: Mean-revert to trailing 8-quarter average
- Operating Margin: Mean-revert to trailing 8-quarter average
- CapEx: Project as % of revenue based on trailing 8-quarter average
- D&A: Project as trailing % of PP&E or revenue
- Tax Rate: Use trailing effective rate or statutory rate
- Share Count: Apply trailing buyback rate (QoQ % change)

Project 4-8 quarters forward.

## Phase 5 — DCF Inputs
Calculate:
- WACC using CAPM: Risk-free rate + (Beta × Equity Risk Premium)
  - Equity Risk Premium: 6.5%
  - Cost of Debt: Interest Expense / Total Debt
  - Tax Rate: Effective tax rate from trailing data
  - Debt/Equity weights from latest balance sheet
- 5-year FCF projections:
  - Annualize quarterly projections
  - FCF = Operating Cash Flow - CapEx
- Terminal value using perpetuity growth (2.5-3%)
- Enterprise Value = PV(5Y FCF) + PV(Terminal Value)
- Equity Value = EV - Net Debt
- Implied Share Price = Equity Value / Shares Outstanding
- Sensitivity matrix: WACC (7 values: base ±2% in 0.5% increments) × terminal growth (6 values: 1.5% to 4.0% in 0.5% increments)

## Phase 6 — Build Excel Model
Create a React artifact that generates an .xlsx file with these tabs:

**Tab 1: Income Statement**
- Rows: all income statement line items
- Columns: historical quarters + projected quarters
- Include YoY growth rows beneath key metrics
- Format as currency/percentage per design system

**Tab 2: Balance Sheet**
- Rows: all balance sheet line items
- Columns: historical quarters + projected quarters
- Include working capital metrics
- Format as currency per design system

**Tab 3: Cash Flow**
- Rows: all cash flow line items
- Columns: historical quarters + projected quarters
- Include FCF calculation
- Format as currency per design system

**Tab 4: Segments**
- Rows: segment revenue and operating income
- Columns: historical quarters + projected quarters
- Include segment margin calculation
- Format as currency/percentage per design system

**Tab 5: KPIs**
- Rows: all company-specific operating KPIs
- Columns: historical quarters + projected quarters
- Include growth rates
- Format per metric type

**Tab 6: Projections**
- Editable assumption inputs (yellow cells):
  - Revenue growth by quarter
  - Gross margin
  - Operating margin
  - CapEx as % of revenue
  - Tax rate
  - Share buyback rate
- Link to other tabs

**Tab 7: DCF**
- WACC calculation breakdown
- 5-year FCF projection (annualized)
- Terminal value calculation
- Sensitivity matrix (WACC × terminal growth)
- Implied share price vs current price

**Tab 8: Summary**
- Company overview (ticker, price, market cap)
- Key metrics summary table
- Valuation summary (DCF range, peer multiples, current valuation)
- Investment highlights

The React artifact should:
1. Import SheetJS: `import * as XLSX from 'xlsx'`
2. Build workbook with XLSX.utils methods
3. Apply cell styling (bold headers, number formats, colors)
4. Generate downloadable .xlsx file
5. Provide download button in the UI

## Output
Present the React artifact to the user with:
- Summary of model structure (8 tabs built)
- Key model outputs:
  - Latest quarterly revenue: [$X.XX billion](https://daloopa.com/src/{id})
  - Projected revenue growth: X.X%
  - DCF implied value: $X.XX (X.X% upside/downside)
  - Peer-implied range: $X.XX - $X.XX
- Note that yellow cells in the Projections tab are editable inputs
- Instructions: Click the download button to save {TICKER}_model.xlsx

All financial figures gathered must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
