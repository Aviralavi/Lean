# TopIndiaMomentum

Self-contained feature package built **on top of** the QuantConnect Lean engine.
Everything we build lives here. **We never modify Lean's open-source code.**

## What this is
A backtest-only, signals-only momentum strategy for Indian equities (NIFTY/BSE):
rank a watchlist by momentum → buy top N → emit SELL signals on drop-out or
stop-loss/take-profit → view results in Lean's report. No brokerage, no real
money — Lean simulates fills; BUY/SELL print to the log as recommendations.

## Layout
```
TopIndiaMomentum/
├── algorithms/   our QCAlgorithm .py files (loaded by path, not compiled)
├── config/       custom Launcher config(s); run with --config
├── docs/         BUILD.md (spec + PR stages) and design notes
└── README.md
```

## Isolation rules
- No edits to Lean dirs (`Algorithm.*`, `Common`, `Engine`, `Launcher`, …).
- Our algorithm is referenced by path via `algorithm-location` in our own
  `config/*.json`; extra modules via `python-additional-paths`. Nothing of ours
  is added to Lean's projects or solution.
- Backtest outputs are git-ignored (see `.gitignore`).

## Run (once code lands in PR 1)
```
dotnet build ../QuantConnect.Lean.sln
cd ../Launcher/bin/Debug
dotnet QuantConnect.Lean.Launcher.dll --config <repo>/TopIndiaMomentum/config/backtest.json
```

See `docs/BUILD.md` for what we're building and the PR-staged plan.
See repo-root `CLAUDE.md` for the fork/sync workflow and guardrails.
