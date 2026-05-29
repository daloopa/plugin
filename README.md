# Daloopa Plugin for Claude Code and Claude Cowork

A plugin for Claude Code and Claude Cowork that adds 21 financial analysis skills powered by [Daloopa](https://daloopa.com) institutional-grade financial data. Works in any project with the Daloopa MCP connection enabled.

## Prerequisites

- **Claude Code or Claude Cowork** - Install Claude Code with `npm install -g @anthropic-ai/claude-code`, or use Claude Cowork in the browser.
- **Daloopa account** - Sign up at [daloopa.com](https://daloopa.com).

## Installation

```bash
# 1. Add the marketplace
claude plugin marketplace add daloopa/daloopa-plugin-claude

# 2. Install the plugin
claude plugin install daloopa
```

## Getting Started

```bash
# 1. Verify the connection
/daloopa:setup

# 2. Run your first analysis
/daloopa:tearsheet AAPL
```

On first use, OAuth may open a browser window for Daloopa login. No API keys are needed.

## Available Commands

| Command | Description | Example |
|---|---|---|
| `/daloopa:setup` | Verify MCP connection and show available skills | `/daloopa:setup` |
| `/daloopa:tearsheet` | Quick one-page company overview | `/daloopa:tearsheet MSFT` |
| `/daloopa:earnings-review` | Full earnings analysis with guidance tracking | `/daloopa:earnings-review AAPL` |
| `/daloopa:earnings-prep` | Pre-earnings preparation report | `/daloopa:earnings-prep NVDA` |
| `/daloopa:earnings-flash` | Rapid first-read earnings flash | `/daloopa:earnings-flash AAPL` |
| `/daloopa:guidance-tracker` | Track management guidance accuracy | `/daloopa:guidance-tracker NVDA` |
| `/daloopa:bull-bear` | Bull/bear/base scenario framework | `/daloopa:bull-bear TSLA` |
| `/daloopa:industry` | Cross-company industry comparison | `/daloopa:industry AAPL MSFT GOOG` |
| `/daloopa:inflection` | Auto-detect metric accelerations/decelerations | `/daloopa:inflection AAPL` |
| `/daloopa:capital-allocation` | Buybacks, dividends, shareholder yield | `/daloopa:capital-allocation MSFT` |
| `/daloopa:dcf` | DCF valuation with sensitivity analysis | `/daloopa:dcf AAPL` |
| `/daloopa:comps` | Trading comparables with peer multiples | `/daloopa:comps AAPL` |
| `/daloopa:precedent-transactions` | Precedent M&A transactions with deal multiples | `/daloopa:precedent-transactions AAPL` |
| `/daloopa:supply-chain` | Interactive supplier and customer dependency dashboard | `/daloopa:supply-chain AAPL` |
| `/daloopa:research-note` | Professional research note | `/daloopa:research-note AAPL` |
| `/daloopa:build-model` | Multi-tab Excel financial model | `/daloopa:build-model AAPL` |
| `/daloopa:comp-sheet` | Industry comp sheet Excel model with operational KPIs | `/daloopa:comp-sheet AAPL` |
| `/daloopa:ib-deck` | Investment banking pitch deck | `/daloopa:ib-deck AAPL` |
| `/daloopa:initiate` | Coverage initiation report plus Excel model | `/daloopa:initiate AAPL` |
| `/daloopa:unit-economics` | Bottoms-up unit economics decomposition | `/daloopa:unit-economics NFLX` |
| `/daloopa:working-capital` | Cash conversion and working capital deep dive | `/daloopa:working-capital AAPL` |

All output is displayed directly in the conversation or saved to `reports/` when a skill produces an HTML, Excel, or deck artifact. You can also ask Claude natural-language questions about a company; the commands are shortcuts for common analysis workflows.

## Data Sources

- **Daloopa MCP** - Institutional-grade financial data from SEC filings, including income statements, balance sheets, cash flow, KPIs, guidance, segment breakdowns, documents, and stock prices.
- **Market data** - Skills use Daloopa stock price data first, then whatever other tools are available in your Claude environment.
- **Consensus estimates** - Included when available; skipped or clearly labeled when unavailable.

Every Daloopa-sourced financial figure should include a citation link back to the original source.

## Related Repos

- This repo is the Claude distribution.
- The Codex and ChatGPT skills distribution lives in the sibling [`daloopa-plugin-codex`](https://github.com/daloopa/daloopa-plugin-codex) repo.
- For enhanced infrastructure and application code, see [github.com/daloopa/investing](https://github.com/daloopa/investing).
