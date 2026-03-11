---
name: build-model
description: Build a multi-tab Excel financial model
argument-hint: TICKER
---

Build a comprehensive Excel financial model (.xlsx) for the company specified by the user: $ARGUMENTS

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow the data access detection logic and design system throughout this skill.

This skill gathers all available financial data and builds a multi-tab Excel model that the user can download directly from their browser.

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
Build forward estimates using inline methodology. Project manually:
- Revenue: guidance + decay to long-term growth
- Margins: mean-revert to trailing averages
- CapEx, D&A, tax rate, share count: trailing trends

Project 4-8 quarters forward.

**Projection Methodology:**
- Calculate trailing 8Q average for each margin metric
- Apply guidance-implied growth rate for next 1-2 quarters, then decay linearly to long-term rate (GDP+1-2%)
- For CapEx, use trailing % of revenue unless guidance suggests otherwise
- For share count, assume trailing buyback rate continues (or zero if no buyback history)
- Tax rate: use trailing effective rate or statutory rate

## Phase 5 — DCF Inputs
Calculate:
- WACC (CAPM: Rf + Beta × ERP; cost of debt from interest/debt)
- 5-year FCF projections (annualized from quarterly)
- Terminal value (perpetuity growth at 2.5-3%)
- Sensitivity matrix: WACC (7 values) × terminal growth (6 values)

**WACC Calculation:**
- Cost of Equity = Risk-Free Rate + Beta × Equity Risk Premium (use 6% ERP)
- Cost of Debt = Interest Expense / Total Debt (use trailing 4Q average)
- WACC = (E/(D+E)) × Cost of Equity + (D/(D+E)) × Cost of Debt × (1 - Tax Rate)

**Terminal Value:**
- Terminal FCF = Final year FCF × (1 + Terminal Growth Rate)
- Terminal Value = Terminal FCF / (WACC - Terminal Growth Rate)

## Phase 6 — Build Excel Model
Generate a React artifact that uses SheetJS (xlsx library) to build and download the .xlsx file directly in the user's browser.

The Excel model must contain these tabs:
1. **Summary** — Key metrics, valuation summary, peer comparison
2. **Income Statement** — All historical and projected P&L line items
3. **Balance Sheet** — All historical and projected balance sheet items
4. **Cash Flow** — OCF, FCF, CapEx, D&A, financing activities
5. **Segments** — Revenue and operating income by segment
6. **KPIs** — All company-specific operating metrics
7. **Projections** — Editable assumption cells (yellow highlight) + forward estimates
8. **DCF** — WACC build-up, FCF projections, terminal value, sensitivity table

**Excel Formatting:**
- Headers: bold, navy background (#1B2A4A), white text
- Financial data: accounting format with $ and parentheses for negatives
- Percentages: 1 decimal place
- Growth rates: below each metric in italics
- Editable cells (Projections tab): yellow fill (#FFF3CD)
- Formulas: use Excel formulas, not static values (e.g., =B5+B6, not hardcoded sum)

**React Artifact Structure:**
```javascript
import React from 'react';
import * as XLSX from 'xlsx';

function FinancialModel() {
  const buildModel = () => {
    // Create workbook
    const wb = XLSX.utils.book_new();
    
    // Build each tab (Summary, Income Statement, Balance Sheet, etc.)
    // Use XLSX.utils.aoa_to_sheet for array-of-arrays data
    // Apply cell styles (bold, colors, number formats)
    
    // Download file
    XLSX.writeFile(wb, '{TICKER}_model.xlsx');
  };
  
  return <button onClick={buildModel}>Download {TICKER} Model</button>;
}
```

## Output
Present to the user:
- The React artifact with a button to download the .xlsx model
- Summary of what tabs were built
- Key model outputs: trailing revenue, projected revenue growth, implied DCF value, peer-implied range
- Remind user that yellow cells in the Projections tab are editable inputs
- Note: Open the downloaded .xlsx in Excel or Google Sheets to view and edit

All financial figures gathered must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
