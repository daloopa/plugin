---
name: research-note
description: Generate a professional Word document research note
argument-hint: TICKER
---

Generate a professional research note for the company specified by the user: $ARGUMENTS

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow the data access detection logic and design system throughout this skill.

This skill gathers comprehensive data and outputs a styled HTML report using the HTML Report Template from design-system.md (full CSS inlined, zero dependencies). Work through each phase sequentially, building up a context object.

## Phase A — Company Setup
Look up the company by ticker using `discover_companies`. Capture:
- `company_id`
- `latest_calendar_quarter` — anchor for all period calculations (see `data-access.md` Section 1.5)
- `latest_fiscal_quarter`
- Firm name for report attribution (default: "Daloopa") — see `data-access.md` Section 4.5

Get current stock price, market cap, shares outstanding, beta, and trading multiples for {TICKER} (see data-access.md Section 2 for how to source market data).

Initialize context: `context = {company_name, ticker, date, price, market_cap, firm_name, ...}`

## Phase B — Core Financials + Cost Structure
Calculate 8 quarters backward from `latest_calendar_quarter`. Pull Income Statement metrics:
- Revenue, Gross Profit, Operating Income, Net Income, Diluted EPS
- EBITDA (compute as Op Income + D&A if not direct, label "(calc.)")
- Operating Expenses (SG&A, R&D where available)

Pull Cash Flow & Balance Sheet:
- Operating Cash Flow, CapEx, Free Cash Flow (OCF - CapEx, label "(calc.)")
- Cash, Total Debt, Net Debt
- D&A

**For every value returned by `get_company_fundamentals`, record its `fundamental_id` (the `id` field).** Store each data point as `{value, fundamental_id}` so citations can be rendered in the final document.

Compute margins and YoY growth rates for each quarter. Every Daloopa-sourced number must include its citation link: `[$X.XX million](https://daloopa.com/src/{fundamental_id})`.

### Cost Structure & Margin Analysis
After the core financial pull, add:

- **COGS driver identification**: Search for cost-related series ("cost of goods", "materials", "manufacturing", "input cost"). Identify 3-5 biggest cost line items and their trends over 8Q.
- **OpEx breakdown**: Pull R&D and SG&A separately. Compute R&D % of revenue and SG&A % of revenue trends over 8Q.
- **Margin driver analysis**: For each major margin (gross, operating, net), identify what's driving expansion or compression — pricing power, cost leverage, mix shift, or one-time items.

## Phase C — KPIs, Segments & Industry Deep Dive
Think about what KPIs matter most for THIS company's business model. Search for:
- Company-specific operating KPIs (subscribers, units, ARPU, retention, etc.)
- Segment revenue breakdown
- Geographic revenue breakdown
- Share count and buyback activity

Pull the same 8 quarters (from `latest_calendar_quarter`).

### Industry-Specific Deep Dive
After the KPI/segment pull, determine the company's sector and apply the relevant analysis template:

- **Manufacturing/Industrial**: Bookings & backlog, book-to-bill ratio, pipeline by geography, capacity utilization
- **SaaS/Technology**: ARR/MRR trajectory, net retention rate, customer cohort analysis, RPO/deferred revenue trends
- **Retail/Consumer**: Same-store sales, store count trajectory, traffic vs ticket decomposition, inventory health
- **Financials/Banks**: NIM trajectory, provision trends, loan growth by category, capital ratios (CET1, TCE)
- **Healthcare/Pharma**: Pipeline summary (drug, indication, phase, milestone), product revenue breakdown, patent cliff timeline
- **Energy**: Production volumes, realized pricing vs benchmark, proved reserves, breakeven analysis

Search for relevant series using `discover_company_series` with sector-appropriate keywords. Pull available data and build the narrative.

## Phase D — Guidance Track Record (follows /guidance-tracker methodology)
Search for guidance series ("guidance", "outlook", "forecast", "estimate", "target").
Pull guidance and corresponding actuals. Apply +1 quarter offset rule.
Compute beat/miss rates and patterns.

## Phase E — What You Need to Believe
Using the financial baseline from Phase B:
- Compute trailing 4Q totals for key metrics (revenue, EBITDA, EPS, FCF)
- Analyze segment-level trends and inflections

Build **falsifiable bull/bear beliefs**:

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

## Phase F — Capital Allocation (follows /capital-allocation methodology)
Pull buyback, dividend, share count, FCF data.
Compute shareholder yield, FCF payout ratio, net leverage.

## Phase G — Valuation (follows /dcf + /comps methodology)

**DCF:**
- Get risk-free rate (see data-access.md Section 2)
- Calculate WACC using CAPM
- Project FCF 5 years (LLM performs calculations directly following the projection methodology: analyze historical FCF trends, identify key drivers, estimate revenue growth trajectory, apply margin assumptions, factor in working capital and CapEx, extend to 5-year horizon)
- Compute terminal value, implied share price, sensitivity table

**Comps:**
- Identify 5-8 peers
- Get peer trading multiples (see data-access.md Section 2)
- If consensus forward estimates are available (data-access.md Section 3), include forward multiples
- Compute implied valuation range from peer multiples

## Phase H — Qualitative Research + News & Catalysts
### SEC Filing Research
Search SEC filings across multiple queries:
- "risk" / "uncertainty" / "challenge" for risk factors
- "growth" / "opportunity" / "expansion" for growth drivers
- "competition" / "market share" for competitive dynamics
- "outlook" / "guidance" for management's forward view
- Company-specific strategic topics (e.g., "AI", "cloud", etc.)

Extract and organize into risks, investment thesis, and company description.

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

## Phase I — Data Tables
Present all chart data in well-formatted tables:

1. **Revenue Trend Table**: 8 quarters of revenue with YoY growth
2. **Margin Trend Table**: Gross margin, operating margin, net margin over 8 quarters
3. **Segment Breakdown Table**: Latest quarter segment revenue with % of total
4. **DCF Sensitivity Table**: Implied prices at various WACC/growth combinations, highlight current price

## Phase J — Synthesis + Tensions + Monitoring
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

## Phase K — Render HTML Report
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

## Output
Present the HTML report directly in your response. Tell the user:
- A 3-4 sentence executive summary of the research note
- Key findings and valuation range
- That they can save the HTML and open it in any browser
