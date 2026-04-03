---
name: build-model
description: "Build a multi-tab Excel financial model (.xlsx) with income statement, balance sheet, cash flow, segments, KPIs, forward projections, and DCF valuation using Daloopa fundamental data and SheetJS. Use when the user asks to build a financial model, create a DCF analysis, generate a pro forma, make a three-statement model, produce a budget spreadsheet, build an Excel valuation workbook, or provides a stock ticker requesting financial projections, forecast, or company valuation."
argument-hint: TICKER
---

Build a comprehensive Excel financial model (.xlsx) for: $ARGUMENTS

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow these throughout.

This skill gathers financial data via Daloopa APIs and builds a multi-tab Excel model as a React artifact using SheetJS that the user can download.

### Data Resolution Strategy

When market data is needed (stock price, peer multiples, risk-free rate), follow this order:
1. MCP market data tools (if available)
2. Web search
3. Sensible defaults (document assumptions)

## Phase 1 — Company Setup

Look up the company by ticker using `discover_companies`. Capture `company_id`, `latest_calendar_quarter`, `latest_fiscal_quarter`, and firm name.

**Checkpoint:** If `discover_companies` returns no results or no `company_id`, stop and tell the user the ticker was not found. Suggest checking the ticker or trying an alternative name.

Get current stock price, market cap, shares outstanding, beta, and trading multiples using the data resolution strategy above.

## Phase 2 — Comprehensive Data Pull

Calculate periods backward from `latest_calendar_quarter`. Target 8-16 quarters.

Pull all standard line items for:
- **Income Statement:** Revenue through diluted EPS, including EBITDA and D&A (compute EBITDA = Op Income + D&A if unavailable)
- **Balance Sheet:** Cash through total equity (current assets, fixed assets, current liabilities, long-term debt, equity)
- **Cash Flow:** Operating CF through FCF, including CapEx, D&A, acquisitions, dividends, buybacks (compute FCF = Op CF - CapEx if unavailable)
- **Segments:** Revenue and operating income by segment
- **KPIs:** All company-specific operating metrics
- **Guidance:** All guidance series and corresponding actuals

**Checkpoint:** Verify at least 4 quarters of income statement data were returned. If fewer, warn the user that limited data may affect projection quality.

## Phase 3 — Market Data & Peers

- Identify 5-8 peers and get trading multiples (use data resolution strategy)
- Get risk-free rate (use data resolution strategy)
- Include NTM estimates for peers if consensus forward estimates are available (see data-access.md Section 3)

## Phase 4 — Projections

Build 4-8 quarters of forward estimates:
- **Revenue:** Latest guidance (if available), then decay to long-term growth rate. Apply quarterly seasonality from trailing data.
- **Gross Margin:** Mean-revert to trailing 8Q average, adjusted for trends/guidance.
- **OpEx:** Project as % of revenue toward trailing averages (R&D and SG&A may differ).
- **CapEx:** % of revenue from trailing 4-8Q average and guidance.
- **D&A:** Trailing average as % of revenue or PP&E.
- **Tax Rate:** Trailing effective rate or guidance.
- **Share Count:** Dilution/buyback from trailing trends and guidance.
- **Working Capital:** DSO, DIO, DPO from trailing averages.

Sum quarterly projections to annual.

**Checkpoint:** Verify projected revenue growth is within a reasonable range (e.g., -20% to +50% YoY). Flag outliers to the user.

## Phase 5 — DCF Inputs

Calculate:
- **WACC:** CAPM cost of equity (Rf + Beta x ERP, ERP = 6.0%). Cost of debt = Interest Expense / Total Debt. WACC = (E/V x Re) + (D/V x Rd x (1 - Tax Rate)).
- **5-year FCF projections** annualized from quarterly (FCF = Op CF - CapEx).
- **Terminal Value:** Perpetuity growth at 2.5-3.0%.
- **Sensitivity Matrix:** WACC (7 values, -3% to +3% from base) x Terminal Growth (6 values, 1.5% to 4.0%).

## Phase 6 — Build Excel Artifact

Generate a React artifact that builds the .xlsx using SheetJS. Use this pattern for sheet creation and formatting:

```jsx
import XLSX from "xlsx";

// Create workbook and add a sheet
const wb = XLSX.utils.book_new();
const wsData = [
  ["Company", ticker, "", "Report Date", new Date().toLocaleDateString()],
  [], // spacer row
  ["", ...quarterHeaders],
  ["Revenue", ...revenueData],
  ["  YoY Growth %", ...revenueGrowth],
];
const ws = XLSX.utils.aoa_to_sheet(wsData);

// Freeze header rows and first column
ws["!freeze"] = { xSplit: 1, ySplit: 3 };
// Set column widths
ws["!cols"] = [{ wch: 24 }, ...quarterHeaders.map(() => ({ wch: 14 }))];

XLSX.utils.book_append_sheet(wb, ws, "Income Statement");
// ... repeat for each tab ...

// Trigger download
const wbout = XLSX.write(wb, { bookType: "xlsx", type: "array" });
const blob = new Blob([wbout], { type: "application/octet-stream" });
const url = URL.createObjectURL(blob);
// Download link with filename: {TICKER}_model.xlsx
```

Create 8 tabs (all use design-system.md formatting, historical + projected columns, frozen panes):

1. **Income Statement** — P&L line items with YoY growth % and margin % sub-rows. Header with company name, ticker, date.
2. **Balance Sheet** — Assets and liabilities with % of Total Assets sub-rows.
3. **Cash Flow** — Operating CF, CapEx, FCF, acquisitions, dividends, buybacks with FCF yield % and CapEx % revenue.
4. **Segments** — Revenue and operating income by segment with % of total and growth rates.
5. **KPIs** — Company-specific operating metrics with YoY growth.
6. **Projections** — Yellow-highlighted editable assumption inputs (growth %, margins, CapEx %, tax rate, buyback rate). Calculated P&L/BS/CF outputs. Methodology commentary box.
7. **DCF** — WACC/terminal growth inputs, 5-year FCF, terminal value, PV calculations, implied share price. Sensitivity table with green/red color scale vs current price.
8. **Summary** — Company overview, market data, valuation summary (DCF implied, peer-implied range, upside/downside %), peer multiples, key outputs.

Include a download button: `{TICKER}_model.xlsx`

**Checkpoint:** Verify the artifact renders without errors before presenting. If SheetJS import fails, suggest the user check their environment supports the xlsx library.

## Output

Present the artifact with:
- Summary of tabs built
- Key outputs: trailing revenue, projected revenue growth, implied DCF value, peer-implied range
- Note that yellow cells in Projections are editable inputs
- Instruction to click download to save the .xlsx file

All Daloopa figures must use citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
