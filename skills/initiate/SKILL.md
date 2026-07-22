---
name: initiate
description: Initiate coverage — generate both research note (.docx) and Excel model (.xlsx)
argument-hint: TICKER
---

Initiate coverage on the company specified by the user: $ARGUMENTS

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow the data access detection logic and design system throughout this skill.

This is the capstone skill that produces both a research note and an Excel model from a single comprehensive data gathering pass.

## Strategy
Rather than running `/research-note` and `/build-model` independently (which would duplicate data gathering), this skill gathers a superset of data once, then renders both outputs.

## Phase 1 — Company Setup
Look up the company by ticker using `discover_companies`. Capture:
- `company_id`
- `latest_calendar_quarter` — anchor for all period calculations (see `data-access.md` Section 1.5)
- `latest_fiscal_quarter`
- Firm name for report attribution (default: "Daloopa") — see `data-access.md` Section 4.5

Get market data (see data-access.md Section 2):
- Current price, market cap, shares outstanding, beta
- Trading multiples (P/E, EV/EBITDA, P/S, P/B)
- Risk-free rate (for DCF)

## Phase 2 — Comprehensive Data Gathering
Follow the `/build-model` skill's Phase 2 data pull (the most comprehensive). Calculate 8-16 quarters backward from `latest_calendar_quarter`. Pull:
- Full Income Statement (Revenue through EPS, including D&A for EBITDA calc)
- Full Balance Sheet (Cash through Equity)
- Full Cash Flow Statement (OCF, CapEx, FCF, Dividends, Buybacks)
- Segment revenue and operating income breakdowns
- Geographic revenue breakdown
- All company-specific operating KPIs
- All guidance series and corresponding actuals
- Share count, buyback amounts

## Phase 3 — Peer Analysis
Identify 5-8 comparable companies.
Get peer trading multiples (see data-access.md Section 2).
If consensus forward estimates are available (data-access.md Section 3), include NTM estimates.
Pull peer fundamentals from Daloopa where available (revenue growth, margins).

## Phase 4 — Projections
Build forward estimates yourself — perform the calculations directly, no external tooling:
- Revenue: guidance + decay to long-term growth
- Margins: mean-revert to trailing averages
- CapEx, D&A, tax rate, share count: trailing trends

## Phase 5 — DCF Valuation
- Calculate WACC (CAPM)
- Project 5-year FCFs
- Terminal value
- Implied share price
- Sensitivity table (WACC × terminal growth)

## Phase 6 — Qualitative Research
Search SEC filings comprehensively:
- Risk factors, growth drivers, competitive dynamics
- Management outlook and guidance language
- Capital allocation strategy
- Company-specific strategic topics
Extract business description, risks (ranked), investment thesis, catalysts.

## Phase 7 — What You Need to Believe
Build falsifiable bull/bear beliefs (follows /research-note methodology):
- 4-6 numbered bull beliefs with evidence and Daloopa citations — each testable in 6 months
- 4-6 numbered bear beliefs with evidence and Daloopa citations — each testable in 6 months
- Valuation math for each side: forward multiple × earnings estimate = price target
- Risk/reward asymmetry assessment (bull upside % vs bear downside %)

## Phase 8 — Synthesis
Write the executive summary, variant perception, and key findings.

Present all trend and sensitivity data as well-formatted tables (no chart images are generated):
1. Revenue trend table — periods as columns, revenue level + YoY growth as rows
2. Margin trend table — periods as columns, gross/operating/net margin as rows
3. Segment mix table — segments as rows, periods as columns, with % of total mix sub-rows
4. DCF sensitivity table — WACC values as rows, terminal growth values as columns, implied share price at each intersection, current price called out for reference

## Phase 9 — Render Both Outputs

**Research Note:**
Compile all gathered data, charts, and narrative sections into a single styled HTML research note using the HTML Report Template from design-system.md (full CSS inlined, zero dependencies). Never output raw markdown — the design system explicitly forbids it. Present it directly in the response.

**Excel Model:**
Generate a React artifact that builds the model in-browser using the SheetJS (`xlsx`) library and triggers a download of `{TICKER}_model.xlsx`. Structure the workbook with one sheet per tab, preserving the same tab structure as the original model:

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
Present both deliverables directly in the response:
- The full styled HTML research note
- The React/SheetJS artifact that downloads `{TICKER}_model.xlsx`

Then provide:
- 3-4 sentence executive summary
- Key valuation range (DCF implied price + comps range)
- Top 3 findings
- Remind the user that shaded cells in the Excel model's Projections tab are editable inputs

All financial figures must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
