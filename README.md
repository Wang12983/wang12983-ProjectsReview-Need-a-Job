# Narrative Exhaustion Quant

A quantitative research system for the **sell-the-news** alpha pattern in meme/retail stocks.

Short thesis: meme stocks run up sharply before earnings events driven by retail FOMO. When reality meets the inflated expectation, the reversal is fast (≤24 hours) and predictable. Enter short at T+1 close, cover at T+2 close.

> **Disclaimer:** This project is for research and paper trading only. No real trades are placed. Past backtest performance does not guarantee future results.

---

## Key Results (Phase 7G — OOS Walk-Forward)

| Metric | Value |
|--------|-------|
| OOS trades | 19 |
| Win rate | **78.9%** |
| Avg net return | **+3.01%** |
| 95% CI | [+1.58%, +4.48%] |
| Sharpe (annualized) | **14.74** |
| p-value | **0.0008** |
| Max drawdown | −0.21% |
| Robustness checks | 7/7 PASS |
| Calendar years | 2022–2026 |
| Unique tickers | 9 |

Signal filter: realized vol ≥ 70%, ADV $50M–$300M, skip hard-to-borrow names.

---

## Research Phases

| Phase | Description | Key Finding |
|-------|-------------|-------------|
| 1–3 | Universe discovery, multi-universe backtest | Meme/retail only. Mega-cap, biotech, small-cap AI all fail. |
| 4 | Options enrichment (IV, call/put ratio, gamma) | Options crowding inconclusive with free data only. |
| 5 | Logistic regression factor analysis | `log_market_cap` dominant (p=0.002). Smaller cap = stronger reversal. |
| 6 | Holding period optimization | Alpha is a 24-hour window. Sharpe collapses 45× from Day 1 to Day 2. |
| 7A–7D | Walk-forward framework, narrative selector | vol≥70% + ADV filter identifies the structural archetype. |
| 7E | Robustness validation | 5/5 checks, 12/12 grid configs positive. |
| 7F | Execution stress tests | 19/19 scenarios positive. HTB skip improves all metrics. |
| 7G | Trade inspection | 19 OOS trades logged. GENUINE SIGNAL confirmed. |
| 8-Lite | Free proxy features for options crowding | `vol_pct_rank` strongest free proxy (r=+0.178). Polygon not worth it at <$500K capital. |
| 9 | NES scoring system | Composite score for capacity scaling. Phase 7G hard filter remains best at $100K. |
| 10 | Paper trading | Live paper trading stack. Scanner + logger + reviewer. |

---

## Strategy Parameters

```
Universe:    25 meme/retail tickers (GME, AMC, RIVN, LCID, AFRM, UPST, …)
Filter:      realized_vol >= 70%  AND  $50M <= ADV <= $300M
Entry:       SHORT at T+1 close (day after earnings report)
Exit:        COVER at T+2 close (1-day hold)
Position:    10% of capital per trade, max 5 concurrent
Stop loss:   none — pure time exit
Borrow:      5% p.a. (easy-to-borrow; HTB names skipped)
Capacity:    ~$2.5M at Phase 7G hard filter
```

---

## Codebase Structure

```
scan_candidates.py       Phase 10: earnings scanner + paper trade logger
review_paper_trades.py   Phase 10: paper trading performance reviewer (read-only)
run_phase7g.py           Phase 7G: trade inspection for strongest config
run_phase7f.py           Phase 7F: execution stress tests
run_phase7e.py           Phase 7E: robustness of narrative selector
run_phase8_lite.py       Phase 8: free proxy features analysis
run_phase9.py            Phase 9: NES scoring system

data/
  universe_expanded.py   25-ticker expanded universe
  event_calendar_builder.py  earnings calendar builder (yfinance)
  price_loader.py        OHLCV loader
  narrative_selector.py  Phase 7G narrative filter

features/
  engineering.py         runup, realized vol, volume spike
  proxy_features.py      Phase 8 proxy features (vol_pct_rank, ATR ratio)

signals/
  sell_the_news.py       signal generation
  nes_scorer.py          Phase 9 NES composite scorer

backtest/
  engine.py              walk-forward backtest engine
  execution.py           slippage, borrow cost, HTB filter
  portfolio.py           portfolio-level position management

reports/
  paper_trades.csv       Phase 10 paper trade ledger
  enriched_events_v2.csv 486 historical events with all features
  scored_events_phase9.csv  NES scores for all events
```

---

## Quick Start

**Reproduce Phase 7G results:**
```bash
python run_phase7g.py
```

**Reproduce robustness checks:**
```bash
python run_phase7e.py
python run_phase7f.py
```

**Run NES scoring system:**
```bash
python run_phase9.py
python run_phase9.py --skip-proxy   # fast run, no proxy features
```

---

## Phase 10 — Paper Trading

Scan for upcoming signals (run Sunday evening):
```bash
python scan_candidates.py
python scan_candidates.py --nes     # include NES scores
```

Log paper entry at T+1 close:
```bash
python scan_candidates.py --log-entry TICKER YYYY-MM-DD
```

Log paper exit at T+2 close:
```bash
python scan_candidates.py --log-exit TICKER YYYY-MM-DD
```

Monthly review:
```bash
python review_paper_trades.py
```

See [PHASE_10_PAPER_TRADING.md](PHASE_10_PAPER_TRADING.md) for the full workflow.

---

## Data Sources

| Source | Usage | Cost |
|--------|-------|------|
| yfinance | OHLCV, earnings dates, market cap | Free |
| Finnhub | Company news (NLP sentiment, news count) | Free tier |
| FINRA RegSho | Short volume ratio | Free |
| Polygon.io | Historical options IV/OI | $200/mo — not used at <$500K |

---

## Key Results Files

```
PHASE_9_RESULTS.md           Full Phase 7G/8/9 summary + Tuesday checklist
PHASE_7G_TRADE_INSPECTION.md All 19 OOS trades with robustness checks
PROJECT_MEMORY.md            Complete research context (phases 1–9)
```

---

## Capital Scaling Guide

| Capital | Recommended Mode | Trades/yr | Sharpe | p-value |
|---------|-----------------|-----------|--------|---------|
| $100K | Phase 7G hard filter | ~4–5 | 14.74 | 0.0008 |
| $500K–$2M | NES ≥ 70 | ~11 | 9.07 | 0.087 |
| $2M–$10M | Top-5 per quarter | ~140 | 2.20 | 0.103 |
| $10M+ | Top-10 per quarter | ~224 | 1.92 | 0.071 |

---

*All backtest results from OOS walk-forward validation (no in-sample fitting on test data). Execution costs included: dynamic slippage, 5%/30% annual borrow, 10 bps exchange fee.*
