---
name: build-model
description: Build a multi-tab Excel financial model
argument-hint: TICKER
---

Build a comprehensive Excel financial model (.xlsx) for the company specified by the user: $ARGUMENTS

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow the data access detection logic and design system throughout this skill.

This skill gathers all available financial data and builds a multi-tab Excel model as a React artifact using SheetJS (xlsx), which the user downloads directly from their browser.

## Phase 1 — Company Setup
Look up the company by ticker using `discover_companies`. Capture:
- `company_id`
- `latest_calendar_quarter` — anchor for all period calculations (see `data-access.md` Section 1.5)
- `latest_fiscal_quarter`
- Firm name for report attribution (default: "Daloopa") — see `data-access.md` Section 4.5

Get current stock price, market cap, shares outstanding, beta, and trading multiples for {TICKER} (see `data-access.md` Section 2 for how to source market data).

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
- Identify 5-8 peers and get their trading multiples (see `data-access.md` Section 2)
- Get risk-free rate (see `data-access.md` Section 2)
- If consensus forward estimates are available (`data-access.md` Section 3), include NTM estimates for peers

## Phase 4 — Projections
Build forward estimates yourself — perform the calculations directly, no external tooling:
- Revenue: guidance + decay to long-term growth
- Margins: mean-revert to trailing averages
- CapEx, D&A, tax rate, share count: trailing trends

Project 4-8 quarters forward.

## Phase 5 — DCF Inputs
Calculate:
- WACC (CAPM: Rf + Beta × ERP; cost of debt from interest/debt)
- 5-year FCF projections (annualized from quarterly)
- Terminal value (perpetuity growth at 2.5-3%)
- Sensitivity matrix: WACC (7 values) × terminal growth (6 values)

## Phase 6 — Build Excel Model
Generate a React artifact that builds the model in-browser using the SheetJS (`xlsx`) library and triggers a download of `{TICKER}_model.xlsx`. Structure the workbook with one sheet per tab, preserving the exact same tab structure as the original model:

- Income Statement
- Balance Sheet
- Cash Flow
- Segments
- KPIs
- Projections
- DCF
- Summary

Assemble all gathered data (company info, market data, periods, historical statements, segments, KPIs, guidance, projections, projection assumptions, DCF outputs, comps) into a single in-memory data object inside the artifact, then use it to populate each sheet via SheetJS (`XLSX.utils.aoa_to_sheet` / `json_to_sheet`, `XLSX.utils.book_append_sheet`, `XLSX.writeFile`).

!!! MANDATORY EXCEL DATAPOINT HYPERLINK FORMAT !!!
- ALWAYS attach a cell hyperlink when writing Daloopa fundamental datapoints to the sheet
- Format: [$<value> <unit>](https://daloopa.com/src/{fundamental_id})
- Example: write `123.4` to cell B5 and set its `l` property (SheetJS cell hyperlink, e.g. `ws['B5'].l = { Target: "https://daloopa.com/src/71667434" }`) so it renders as [$123.4 million](https://daloopa.com/src/71667434)
- Set the hyperlink on that exact cell; do not write the URL into a neighboring cell or as plain text
- NEVER write a Daloopa-sourced numeric datapoint without its cell hyperlink
- Every Daloopa datapoint cell MUST include the clickable source link
- Do NOT add datapoint hyperlinks to headers, labels, blank cells, or unsourced formulas

Mark projected/editable cells (e.g. the Projections tab assumptions) with distinct fill styling so the user can identify and adjust inputs after download.

If SheetJS generation fails for any reason, report the error clearly so the user can retry.

## Output
Tell the user:
- That the .xlsx model downloaded as `{TICKER}_model.xlsx`
- Summary of what tabs were built
- Key model outputs: trailing revenue, projected revenue growth, implied DCF value, peer-implied range
- Remind the user that shaded cells in the Projections tab are editable inputs

All financial figures gathered must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
