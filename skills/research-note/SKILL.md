---
name: research-note
description: Generate a professional Word document research note
argument-hint: TICKER
---

Generate a professional research note for the company specified by the user: $ARGUMENTS

**Before starting, read `data-access.md` for data access methods and `design-system.md` for formatting conventions.** Follow the data access detection logic and design system throughout this skill.

This skill gathers comprehensive data and presents a structured research note as formatted markdown directly in the response.

## Phase A — Company Setup
Look up the company by ticker using `discover_companies`. Capture:
- `company_id`
- `latest_calendar_quarter` — anchor for all period calculations (see `data-access.md` Section 1.5)
- `latest_fiscal_quarter`
- Firm name for report attribution (default: "Daloopa") — see `data-access.md` Section 4.5

Get current stock price, market cap, shares outstanding, beta, and trading multiples for {TICKER} (see data-access.md Section 2 for how to source market data).

Initialize context: `{company_name, ticker, date, price, market_cap, firm_name, ...}`

## Phase B — Core Financials + Cost Structure
Calculate 8 quarters backward from `latest_calendar_quarter`. Pull Income Statement metrics:
- Revenue, Gross Profit, Operating Income, Net Income, Diluted EPS
- EBITDA (compute as Op Income + D&A if not direct, label "(calc.)")
- Operating Expenses (SG&A, R&D where available)

Pull Cash Flow & Balance Sheet:
- Operating Cash Flow, CapEx, Free Cash Flow (OCF - CapEx, label "(calc.)")
- Cash, Total Debt, Net Debt
- D&A

**For every value returned by `get_company_fundamentals`, record its `fundamental_id` (the `id` field).** Store each data point with its citation so citations can be rendered in the final document.

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
- Project FCF 5 years — perform calculations directly
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

Extract and organize into:
- Risks — ranked list of risks with impact/probability
- Investment thesis — variant perception, thesis pillars, catalysts
- Company description — 2-3 sentence business description

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

## Phase I — Synthesis + Tensions + Monitoring
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

## Phase J — Present Research Note
Present the research note as structured markdown with the following sections:

### Cover Page
- Company Name, Ticker, Report Date
- Current Price, Market Cap
- Firm Attribution (default: "Daloopa")

### Five Key Tensions
Numbered list, alternating bull/bear

### Executive Summary
- 3-4 sentence TL;DR with clear directional view
- Key metrics table (metric, current value, vs prior period)

### Investment Thesis & Variant Perception
- Investment thesis narrative
- What the market thinks vs what the data shows
- Company description

### Recent News & Catalysts
- News timeline (reverse chronological, last 6-12 months)
- Forward catalysts (organized by timeframe: near/medium/long-term)
- Policy backdrop (if material)

### Financial Analysis
- 8-quarter financial table (columns = periods, rows = metrics)
- Include Revenue, Gross Profit, Operating Income, Net Income, EPS, EBITDA, margins, YoY growth rows
- Cost structure & margin analysis narrative
- OpEx breakdown table (R&D, SG&A, % of revenue trends)

### Segment & Geographic Analysis
- Segment revenue table (if available)
- Geographic revenue table (if available)
- KPI table (company-specific operating metrics)

### Industry-Specific Deep Dive
Sector-appropriate analysis based on Phase C template

### Guidance Track Record
- Guidance accuracy table (if guidance data available)
- Beat/miss analysis narrative

### What You Need to Believe
- Bull beliefs (numbered, falsifiable, with evidence)
- Bull price target + valuation math
- Bear beliefs (numbered, falsifiable, with evidence)
- Bear price target + valuation math
- Risk/reward assessment

### Capital Allocation
- Shareholder yield, FCF payout ratio, net leverage analysis
- Buyback & dividend trends
- Share count table

### Valuation
**DCF Analysis:**
- WACC calculation
- 5-year FCF projections
- Terminal value, implied share price
- Sensitivity table (WACC vs terminal growth rate)

**Comps Analysis:**
- Peer trading multiples table
- Implied valuation range from peer multiples
- Forward multiples (if consensus available)

### Risks
- Ranked list of risks with impact/probability assessment

### Monitoring Framework
- Quantitative monitors (numbered list with thresholds)
- Qualitative monitors (numbered list)

### Appendix
- Methodology notes
- Data sources and limitations

## Output
Present the research note directly as formatted markdown in your response. Use clear section headers, professional table formatting, and maintain all Daloopa citations.

**Citation enforcement:** Every financial figure from Daloopa must use citation format: `[$X.XX million](https://daloopa.com/src/{fundamental_id})`. If a number came from `get_company_fundamentals`, it must have a citation link. No exceptions.

After presenting the full research note, provide:
- A 3-4 sentence executive summary
- Key findings and valuation range
