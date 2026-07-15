# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this repo is

A **private fork of [QuantConnect Lean](https://github.com/QuantConnect/Lean)**
— an open-source (Apache-2.0), event-driven algorithmic-trading **engine**. Lean
runs a strategy *you write* (a `QCAlgorithm`); it is **not** a stock-picker app
and ships **no** web dashboard. C# core + Python algorithm support.

- `origin`   → our fork (push here)
- `upstream` → `QuantConnect/Lean` (pull engine updates; never push)
- Work branch: `india-momentum`. **`master` stays a pure mirror of upstream.**

## What we are building (on top of Lean)

**TopIndiaMomentum** — a backtest-only, signals-only momentum strategy for
Indian equities (NIFTY/BSE):

1. Rank a watchlist of India stocks by momentum → output the **top N to buy**.
2. Hold picks, record entries, emit a **SELL signal** when a name drops out of
   the top N **or** hits stop-loss / take-profit.
3. View results in Lean's post-backtest **HTML report** (the "dashboard").

No brokerage, no real money — Lean simulates fills in backtest; BUY/SELL are
logged as recommendations the user acts on manually.

Full spec + PR stages: `TopIndiaMomentum/docs/BUILD.md`.

## THE GOLDEN RULE — do not touch open-source code

All our work lives **only** in `TopIndiaMomentum/`. Never modify Lean's own dirs
(`Algorithm.*`, `Algorithm.Framework`, `Common`, `Engine`, `Launcher`,
`Indicators`, `Data`, `Report`, `Research`, `Tests`, `ToolBox`, the `.sln`, etc.)
and never edit tracked shared files like `Launcher/config.json`. Keeping our
changes to new files in one folder makes upstream syncs conflict-free.

Isolation mechanics (no engine edits needed):
- Reference our algorithm by **path** via `algorithm-location` in our **own**
  config under `TopIndiaMomentum/config/*.json`; run with
  `--config TopIndiaMomentum/config/backtest.json`.
- Add our module dirs via `python-additional-paths` in that config.
- The one exception is this root `CLAUDE.md` — meta guidance, a new file, adds no
  code to the engine.

## Package layout

```
TopIndiaMomentum/
├── algorithms/   our QCAlgorithm .py files (loaded by path)
├── config/       custom Launcher config(s)
├── docs/         BUILD.md (spec + PR stages)
├── .gitignore    ignores backtest outputs
└── README.md
```

## Data reality (important constraint)

Lean ships free **US** sample data but only **3 India sample stocks**
(`YESBANK`, `JUNIORBEES`, `CCCL`) + `nifty50` index, and **no** India
fundamental/universe files. So:
- We use a **fixed watchlist ranked in-algorithm**, not `AddUniverse`.
- MVP runs on the 3 sample symbols to prove the pipeline.
- Real NIFTY 50 backtests require supplying India history in Lean's zip/CSV
  format under `Data/equity/india/daily/` (data you have rights to, or
  QuantConnect's paid India dataset). Adding data files is allowed; editing
  engine code is not.

India is otherwise supported: `Market.India`, `set_account_currency("INR")`,
`IndiaOrderProperties(Exchange.NSE)`. Patterns to copy:
`Algorithm.Python/BasicTemplateIndiaAlgorithm.py` and
`Algorithm.Python/CoarseFundamentalTop3Algorithm.py`.

## Run (backtest, free, local data)

```
dotnet build QuantConnect.Lean.sln
cd Launcher/bin/Debug
dotnet QuantConnect.Lean.Launcher.dll --config <repo>/TopIndiaMomentum/config/backtest.json
```

## Sync with upstream (periodic)

```
git fetch upstream
git switch master && git merge --ff-only upstream/master
git push origin master
git switch india-momentum && git rebase master
```
`master` fast-forwards cleanly (never diverges); our new-files-only branch
rebases on top conflict-free.

## Conventions

- New code only, inside `TopIndiaMomentum/`. If a task seems to need an engine
  edit, stop and reconsider — there is almost always a config/path-based way.
- Commit messages end with the Co-Authored-By trailer.
- Keep Lean's `LICENSE` intact (Apache-2.0 permits our private modified copy).
