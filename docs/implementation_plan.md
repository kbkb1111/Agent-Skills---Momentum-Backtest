# Implementation Plan - Momentum Backtest

## Approach
Copy the structure of `multi-instrument-channel-breakout-backtest/mib_backtest.py` (loading, calendar alignment, metrics, dark tearsheet) and replace the channel-breakout engine with a rank-and-switch momentum engine. The breakout skill is untouched.

## Components in `mom_backtest.py`
1. `load` / `collect` / `align` - read files, validate, build union calendar (reused, needs only date/open/close).
2. `signal_days` - flags last trading day of each ISO week / month.
3. `momentum_scores` - for each signal day, return over lookback using each asset's full history (`Series.asof`), optional skip.
4. `run` - on signal day rank, pick top N (+ absolute filter), set pending buys/sells; next open fills sells then buys; benchmark rebalanced on the same days.
5. Metrics / charts / HTML / PDF - reused, plus holdings timeline and signal history table.
6. `main` - arguments, validation, data-frequency guard, outputs.

## Design decisions
- Lookback unit follows the signal frequency (weeks or months) so it works the same on daily/weekly/monthly files.
- Winners that stay are not trimmed (less trading); buys use cash / empty slots.
- Benchmark is invested from day one; strategy waits in cash until the first ranking (shown as a warning).

- Fill price: `--fill next-open` (default) or `signal-close`. In signal-close mode `load` only needs date+close (open is set equal to close) and `run` fills right after the signal step on the same bar, using a nested `fill(t)` helper.

- Rebalancing: `rebalance(t)` inside `run` executes after the sells/buys of the signal's fill (flag `rb_due` set on the signal day). Valued at the fill price; trims then top-ups; orders logged in `rebs` and added to the position's cost/proceeds.

## Changelog
- **[10/11/2026]** Added `--rebalance on-signal`. Regression-tested (off = identical results on synthetic and real data); with it on, holdings checked equal within about 0.1% after the first rebalance.
- **[10/11/2026]** Added `--fill signal-close` option (see PRD). Regression-tested: default mode results unchanged on the synthetic data.
- **[10/11/2026]** Plan written and implemented; tested on synthetic data (hand-checked first ranking, fill price and share count).
