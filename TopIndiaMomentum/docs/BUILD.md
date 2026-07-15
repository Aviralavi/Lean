# Top-India-Momentum — Build Spec

## What we are building

A **backtest-only, signals-only** trading strategy on the Lean engine that:

1. Ranks a watchlist of Indian (NIFTY/BSE) stocks by a profit metric (momentum)
   and outputs the **top N to buy**.
2. Holds those picks, records each entry, and emits a **SELL signal** when a name
   drops out of the top N **or** hits a stop-loss / take-profit.
3. Produces a **dashboard** = Lean's post-backtest HTML report (equity curve,
   drawdown, trade list).

**No brokerage, no real money.** Lean simulates fills in backtest; BUY/SELL are
printed to the log as recommendations you act on manually.

### Key facts / constraints
- Lean is an engine that runs a strategy *you write* — no built-in stock-picker
  or web UI.
- India is supported: `Market.India`, INR currency, `IndiaOrderProperties`,
  market hours all ship.
- **Data gap:** only 3 India sample stocks ship (`YESBANK`, `JUNIORBEES`,
  `CCCL`) + `nifty50` index. India fundamental/universe files do NOT ship, so we
  use a **fixed watchlist ranked in-algorithm**, not `AddUniverse`. Real NIFTY 50
  requires supplying your own India history in Lean's zip/CSV format.

### Patterns to reuse (existing files)
- `Algorithm.Python/BasicTemplateIndiaAlgorithm.py` — India setup.
- `Algorithm.Python/CoarseFundamentalTop3Algorithm.py` — rank → top N → liquidate shape.
- `Launcher/config.json` — point engine at a Python algo + local data.
- `Report/` — HTML/PDF report generator (the dashboard).
- `Data/equity/india/daily/cccl.zip` — the data format to mirror when adding stocks.

---

## PR stages

### PR 1 — Skeleton algo runs on sample data
**Goal:** end-to-end pipeline green today, no real trading logic yet.
- Add `Algorithm.Python/TopIndiaMomentumAlgorithm.py`: INR currency, dates, cash,
  `add_equity` for the 3 sample symbols (`Market.INDIA`, `Resolution.DAILY`),
  `IndiaOrderProperties(Exchange.NSE)`, simple buy-and-hold in `on_data`.
- Point `Launcher/config.json` at it (`algorithm-language: Python`,
  `algorithm-location`, `environment: backtesting`).
- **Done when:** `dotnet build` + Launcher run completes with a simulated FILL.

### PR 2 — Momentum ranking + "top N to buy"
**Goal:** produce the buy list.
- Compute per-symbol momentum (N-day return via `self.momp` or `self.history`).
- Monthly rebalance: sort watchlist desc, take top N, log each as `BUY`,
  `set_holdings(sym, 1/N)`.
- **Done when:** log shows dated top-N BUY lines and equal-weight allocation.

### PR 3 — Sell rules + entry history
**Goal:** the "call to sell".
- Store entry price per held symbol (dict).
- SELL + `liquidate` when: dropped out of top N **OR** stop-loss / take-profit hit.
- Log every exit as `SELL` with entry price and reason.
- **Done when:** forced case (tight stop) produces a `SELL`+`Liquidate` log line.

### PR 4 — Dashboard / report
**Goal:** shareable results view.
- Run `Report/` project on the backtest result JSON → HTML report.
- Verify equity curve, drawdown, and a BUY/SELL trades table render.
- **Done when:** report HTML opens and matches logged signals.

### PR 5 — Scale to real NIFTY 50 (optional, needs data)
**Goal:** rank the actual index, not just 3 symbols.
- Add India daily bars under `Data/equity/india/daily/<symbol>.zip` (mirror
  `cccl.zip`) for the NIFTY 50 constituents, sourced from data you have rights to
  (or QuantConnect's paid India dataset via Lean CLI).
- Expand the watchlist to 50 tickers — no logic change.
- **Done when:** backtest ranks across the full list with no missing-data errors.

---

## How to run (free, local)
```
dotnet build QuantConnect.Lean.sln
cd Launcher/bin/Debug
dotnet QuantConnect.Lean.Launcher.dll
```
`config.json` uses local `Data/` so no paid feed is needed for the sample-data MVP.

## Caveats
- "Top stocks" = momentum rank over a watchlist you define, not a market-wide screener.
- Momentum is a profit proxy, not advice; backtest results ≠ future returns.
- Signals-only: you place any real orders yourself, outside Lean.
