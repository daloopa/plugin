---
name: initiate
description: Initiate coverage — generate both research note and Excel model
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
Calculate 8-16 quarters backward from `latest_calendar_quarter`. Pull:
- Full Income Statement (Revenue through EPS, including D&A for EBITDA calc)
- Full Balance Sheet (Cash through Equity)
- Full Cash Flow Statement (OCF, CapEx, FCF, Dividends, Buybacks)
- Segment revenue and operating income breakdowns
- Geographic revenue breakdown
- All company-specific operating KPIs
- All guidance series and corresponding actuals
- Share count, buyback amounts

**For every value returned by `get_company_fundamentals`, record its `fundamental_id` (the `id` field).** Store each data point with its citation so citations can be rendered in both outputs.

## Phase 3 — Peer Analysis
Identify 5-8 comparable companies.
Get peer trading multiples (see data-access.md Section 2).
If consensus forward estimates are available (data-access.md Section 3), include NTM estimates.
Pull peer fundamentals from Daloopa where available (revenue growth, margins).

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

## Phase 5 — DCF Valuation
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

## Phase 6 — Qualitative Research + News & Catalysts
### SEC Filing Research
Search SEC filings comprehensively:
- Risk factors, growth drivers, competitive dynamics
- Management outlook and guidance language
- Capital allocation strategy
- Company-specific strategic topics
Extract business description, risks (ranked), investment thesis, catalysts.

### News & Catalysts via WebSearch
Run 4 WebSearch queries to gather recent external context:
1. `"{TICKER} {company_name} news {year}"` — recent headlines and developments
2. `"{TICKER} analyst upgrade downgrade price target"` — sell-side sentiment shifts
3. `"{TICKER} catalysts risks"` — forward-looking events and risk factors
4. `"{company_name} industry outlook {sector}"` — macro and industry trends

Organize results into three sections:

- **News Timeline** — 6-10 key events from the last 6-12 months in reverse chronological order. Each event: date, headline, 1-sentence impact, sentiment tag (Positive / Negative / Mixed / Upcoming). Format as a numbered list.

- **Forward Catalysts** — Organized by timeframe:
  - **Near-term (0-3 months, HIGH priority)**: earnings dates, product launches, regulatory decisions
  - **Medium-term (3-12 months, MEDIUM priority)**: strategic milestones, contract renewals, industry events
  - **Long-term (1-3 years, LOW priority)**: secular trends, market expansion, competitive dynamics

- **Policy Backdrop** — Macro/regulatory context affecting the company. Tariffs, regulation, interest rates, sector-specific policy. Omit if not material.

## Phase 7 — Cost Structure & Industry Deep Dive
### Cost Structure & Margin Analysis
- **COGS driver identification**: Search for cost-related series ("cost of goods", "materials", "manufacturing", "input cost"). Identify 3-5 biggest cost line items and their trends over 8Q.
- **OpEx breakdown**: Pull R&D and SG&A separately. Compute R&D % of revenue and SG&A % of revenue trends over 8Q.
- **Margin driver analysis**: For each major margin (gross, operating, net), identify what's driving expansion or compression — pricing power, cost leverage, mix shift, or one-time items.

### Industry-Specific Deep Dive
Determine the company's sector and apply the relevant analysis template:

- **Manufacturing/Industrial**: Bookings & backlog, book-to-bill ratio, pipeline by geography, capacity utilization
- **SaaS/Technology**: ARR/MRR trajectory, net retention rate, customer cohort analysis, RPO/deferred revenue trends
- **Retail/Consumer**: Same-store sales, store count trajectory, traffic vs ticket decomposition, inventory health
- **Financials/Banks**: NIM trajectory, provision trends, loan growth by category, capital ratios (CET1, TCE)
- **Healthcare/Pharma**: Pipeline summary (drug, indication, phase, milestone), product revenue breakdown, patent cliff timeline
- **Energy**: Production volumes, realized pricing vs benchmark, proved reserves, breakeven analysis

Search for relevant series using `discover_company_series` with sector-appropriate keywords. Pull available data and build the narrative.

## Phase 8 — Guidance Track Record
Search for guidance series ("guidance", "outlook", "forecast", "estimate", "target").
Pull guidance and corresponding actuals. Apply +1 quarter offset rule.
Compute beat/miss rates and patterns.

## Phase 9 — What You Need to Believe
Build falsifiable bull/bear beliefs:

### Bull Beliefs (To Go Long)
Write 4-6 numbered beliefs, each with:
- One **bold statement** (the belief itself)
- 2-3 sentences of **evidence** with Daloopa citations supporting why this could be true
- Each belief must be **falsifiable** — testable with observable data within 6 months

Example format: "1. **Revenue growth re-accelerates to 15%+ as AI monetization scales.** Cloud segment grew [$X.Xbn](link) last quarter, up X% YoY, with management noting..."

### Bear Beliefs (To Go Short)
Same format — 4-6 numbered falsifiable beliefs with evidence for the downside case.

### Valuation Math
For each side:
- Bull target: forward multiple × forward earnings estimate = price target. Show the math.
- Bear target: same structure with bear-case multiple and earnings.

### Risk/Reward Assessment
- Compare bull upside % vs bear downside % from current price
- If asymmetry is significant (e.g., 30% upside vs 40% downside), flag it explicitly
- State which side has the better risk/reward and why

## Phase 10 — Capital Allocation
Pull buyback, dividend, share count, FCF data.
Compute shareholder yield, FCF payout ratio, net leverage.

## Phase 11 — Synthesis + Tensions + Monitoring
This is the most judgment-intensive step. Be honest and critical — the reader is a professional investor who needs your real assessment, not a balanced summary.

### Core Synthesis
Write:
- **Executive Summary**: 3-4 sentence TL;DR covering current state, key thesis, valuation view. Include a clear directional view — is this stock attractive, fairly valued, or overvalued at the current price?
- **Variant Perception**: What does the market think vs what do you see in the data? Where is the consensus wrong? If you agree with consensus, say that too — but explain what could change.
- **Key Findings**: Top 3-5 most notable data points or trends — prioritize what changes the investment thesis, not just what's interesting
- **Red Flags & Concerns**: Any quality-of-earnings issues, sustainability questions, or risks the market may be underpricing

### Five Key Tensions
Identify the 5 most critical bull/bear debates for this stock. Each tension is a single line that frames both sides. Alternate between bullish-leaning and bearish-leaning tensions. Every tension must reference a specific data point from the analysis.

Format as a numbered list:
1. "[Bullish factor] vs [Bearish factor]" — cite the specific metric
2. "[Bearish factor] vs [Bullish factor]" — cite the specific metric
...etc.

### Monitoring Framework
Build two monitoring lists for ongoing tracking:

**Quantitative Monitors** — 5-7 specific metrics with explicit thresholds:
- Format: "Metric: current value → bull threshold / bear threshold"
- Example: "Gross Margin: 45.2% → above 46% confirms pricing power / below 43% signals cost pressure"

**Qualitative Monitors** — 5-7 factors to watch:
- Management tone shifts on earnings calls
- Competitive dynamics (new entrants, pricing pressure)
- Regulatory developments
- Customer concentration changes
- Capital allocation pivots

## Phase 12 — Render Both Outputs

### Research Note (Structured Markdown)
Present the research note as formatted markdown with the following sections:

**Cover Page**
- Company Name, Ticker, Report Date
- Current Price, Market Cap
- Firm Attribution (default: "Daloopa")

**Five Key Tensions**
Numbered list, alternating bull/bear

**Executive Summary**
- 3-4 sentence TL;DR with clear directional view
- Key metrics table (metric, current value, vs prior period)

**Investment Thesis & Variant Perception**
- Investment thesis narrative
- What the market thinks vs what the data shows
- Company description

**Recent News & Catalysts**
- News timeline (reverse chronological, last 6-12 months)
- Forward catalysts (organized by timeframe: near/medium/long-term)
- Policy backdrop (if material)

**Financial Analysis**
- 8-quarter financial table (columns = periods, rows = metrics)
- Include Revenue, Gross Profit, Operating Income, Net Income, EPS, EBITDA, margins, YoY growth rows
- Cost structure & margin analysis narrative
- OpEx breakdown table (R&D, SG&A, % of revenue trends)

**Segment & Geographic Analysis**
- Segment revenue table (if available)
- Geographic revenue table (if available)
- KPI table (company-specific operating metrics)

**Industry-Specific Deep Dive**
Sector-appropriate analysis based on Phase 7 template

**Guidance Track Record**
- Guidance accuracy table (if guidance data available)
- Beat/miss analysis narrative

**What You Need to Believe**
- Bull beliefs (numbered, falsifiable, with evidence)
- Bull price target + valuation math
- Bear beliefs (numbered, falsifiable, with evidence)
- Bear price target + valuation math
- Risk/reward assessment

**Capital Allocation**
- Shareholder yield, FCF payout ratio, net leverage analysis
- Buyback & dividend trends
- Share count table

**Valuation**
**DCF Analysis:**
- WACC calculation
- 5-year FCF projections
- Terminal value, implied share price
- Sensitivity table (WACC vs terminal growth rate)

**Comps Analysis:**
- Peer trading multiples table
- Implied valuation range from peer multiples
- Forward multiples (if consensus available)

**Risks**
- Ranked list of risks with impact/probability assessment

**Monitoring Framework**
- Quantitative monitors (numbered list with thresholds)
- Qualitative monitors (numbered list)

**Appendix**
- Methodology notes
- Data sources and limitations

### Excel Model (React Artifact with SheetJS)
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
1. The complete research note as formatted markdown
2. The React artifact with a button to download the .xlsx model
3. Summary of what was generated:
   - 3-4 sentence executive summary
   - Key valuation range (DCF implied price + comps range)
   - Top 3 findings
4. Remind user that yellow cells in the Excel model's Projections tab are editable inputs

**Citation enforcement:** Every financial figure from Daloopa must use citation format: `[$X.XX million](https://daloopa.com/src/{fundamental_id})`. If a number came from `get_company_fundamentals`, it must have a citation link. No exceptions.
