---
name: initiate
description: Initiate coverage — generate both research note (HTML) and Excel model (.xlsx)
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

**For every value returned by `get_company_fundamentals`, record its `fundamental_id` (the `id` field).** Store each data point as `{value, fundamental_id}` so citations can be rendered in both outputs.

## Phase 3 — Peer Analysis
Identify 5-8 comparable companies.
Get peer trading multiples (see data-access.md Section 2).
If consensus forward estimates are available (data-access.md Section 3), include NTM estimates.
Pull peer fundamentals from Daloopa where available (revenue growth, margins).

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

## Phase 5 — DCF Valuation
- Calculate WACC using CAPM: Risk-free rate + (Beta × Equity Risk Premium)
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

Organize results into:

- **News Timeline**: 6-10 key events from the last 6-12 months in reverse chronological order. Each event: date, headline, 1-sentence impact, sentiment tag (Positive / Negative / Mixed / Upcoming). Format as a numbered list.

- **Forward Catalysts**: Organized by timeframe:
  - **Near-term (0-3 months, HIGH priority)**: earnings dates, product launches, regulatory decisions
  - **Medium-term (3-12 months, MEDIUM priority)**: strategic milestones, contract renewals, industry events
  - **Long-term (1-3 years, LOW priority)**: secular trends, market expansion, competitive dynamics

- **Policy Backdrop**: Macro/regulatory context affecting the company. Tariffs, regulation, interest rates, sector-specific policy. Leave empty if not material.

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

## Phase 8 — What You Need to Believe
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

## Phase 9 — Synthesis + Tensions + Monitoring
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

## Phase 10 — Render Both Outputs

### Research Note (HTML Report)
Using the HTML Report Template from design-system.md (full CSS inlined in the design system), generate a complete HTML report with the following structure:

1. **Cover Section**: Company name, ticker, date, current price, market cap, five key tensions
2. **Executive Summary**: Key metrics table, executive summary, variant perception
3. **Investment Thesis & Company Overview**: Investment thesis, company description
4. **News & Catalysts**: News timeline, forward catalysts, policy backdrop
5. **Financial Analysis**: Revenue trend table, financial metrics table, margin trend table, cost structure analysis, OpEx breakdown, segment breakdown, geographic breakdown, share count history
6. **Industry Deep Dive**: Sector-specific analysis
7. **Guidance Track Record**: Guidance table and commentary (if available)
8. **What You Need to Believe**: Bull beliefs with valuation math, bear beliefs with valuation math, risk/reward assessment
9. **Capital Allocation**: Commentary on buybacks, dividends, shareholder yield
10. **Valuation**: DCF summary with sensitivity table, comps commentary with peer multiples
11. **Risks**: Risk summary from SEC filings
12. **Monitoring Framework**: Quantitative monitors, qualitative monitors
13. **Appendix**: Methodology notes

**Citation enforcement:** Every financial figure from Daloopa must use citation format: `[$X.XX million](https://daloopa.com/src/{fundamental_id})`. If a number came from `get_company_fundamentals`, it must have a citation link. No exceptions.

All tables follow the standard financial analysis format: columns = time periods, rows = metrics.

Use the full CSS from design-system.md's HTML Report Template section. The output is a complete, standalone HTML file with all styles inlined.

### Excel Model (React Artifact with SheetJS)
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
Tell the user:
- **Research Note (HTML)**: 3-4 sentence executive summary, key findings, valuation range, note that they can save the HTML and open it in any browser
- **Excel Model**: Summary of model structure (8 tabs), key model outputs:
  - Latest quarterly revenue: [$X.XX billion](https://daloopa.com/src/{id})
  - Projected revenue growth: X.X%
  - DCF implied value: $X.XX (X.X% upside/downside)
  - Peer-implied range: $X.XX - $X.XX
- Note that yellow cells in the Projections tab are editable inputs
- Instructions: Click the download button to save {TICKER}_model.xlsx

Present both the HTML report and the React artifact directly in your response.

All financial figures must use Daloopa citation format: [$X.XX million](https://daloopa.com/src/{fundamental_id})
