# TradingAgents — Research Improvements

> Fork of [TauricResearch/TradingAgents](https://github.com/TauricResearch/TradingAgents),
> licensed under Apache-2.0.
> Original paper: [TradingAgents: Multi-Agents LLM Financial Trading Framework](https://arxiv.org/abs/2412.20138).

## Roadmap

| # | Status | Improvement | Upstream issue | PR |
|---|--------|-------------|----------------|----|
| A1 | ✅ Done | Fix look-ahead bias in social-sentiment pipeline | [#1220](https://github.com/TauricResearch/TradingAgents/issues/1220) | [#1232](https://github.com/TauricResearch/TradingAgents/pull/1232) |
| A2 | ✅ Done | Reproducible backtesting / evaluation harness | [#119](https://github.com/TauricResearch/TradingAgents/issues/119) | [#1234](https://github.com/TauricResearch/TradingAgents/pull/1234) |
| A3 | ✅ Done | Fix first-speaker fabrication in multi-agent debates | [#1176](https://github.com/TauricResearch/TradingAgents/issues/1176) | [#1233](https://github.com/TauricResearch/TradingAgents/pull/1233) |

## A1: Look-ahead bias fix
- **Problem**: historical runs fed today's StockTwits/Reddit posts into the
  sentiment prompt as if they covered the requested period.
- **Fix**: both fetchers now filter by the analysis window; empty results
  degrade to explicit placeholders; prompt labels show the real window.
- **Validation**: 9 new regression tests; full suite 576 passed / 2 skipped.

## A2: Backtest harness

- **Problem**: the paper reports backtest results but the repository ships no
  tool to reproduce them (#119).
- **Fix**: a ticker x date harness (`tradingagents/evaluation/backtest.py`
  plus a CLI) that runs the full graph, records the 5-tier rating, resolves
  realized return and alpha vs the regional benchmark, and writes a CSV +
  markdown summary; memory log disabled by default; one-bad-cell isolation.
- **Validation**: 8 tests (dates, summary stats, CSV output, failure
  isolation, memory-log opt-out).

### A2: Backtest results

7-date NVDA pilot (2024-01-05 to 2024-03-29, weekly grid, 5-day holding):

| Rating | Runs | Avg raw return | Avg alpha | Directional hit rate |
|---|---:|---:|---:|---:|
| Overweight | 2 | +5.81% | +4.60% | 100% |
| Hold | 4 | +4.77% | +4.11% | n/a |
| Underweight | 1 | +11.43% | +9.56% | 0% |
| **Overall** | 7 | +6.02% | +5.03% | **67% (2/3)** |

The harness resolves realized returns and alpha vs SPY for every run and
quantifies directional accuracy of the 5-tier ratings; failures are recorded
rather than discarded (the single Underweight call was a documented miss).

## A3: Debate fabrication fix

- **Problem**: the first speaker in each debate was prompted to rebut an
  opponent argument that did not exist yet, causing models to fabricate the
  opponent's position (#1176).
- **Fix**: opening speakers get an explicit "present your own case"
  instruction; rebuttal framing (including the "Last bear/bull argument"
  line) is only used once the opponent has actually spoken. Same branch for
  the risk debate (Aggressive / Conservative / Neutral).
- **Validation**: 9 regression tests covering opening and rebuttal modes for
  all five debate agents.
