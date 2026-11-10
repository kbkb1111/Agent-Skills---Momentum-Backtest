---
name: momentum-backtest
description: Backtests a long-only momentum rotation strategy across SEVERAL stocks/ETFs/assets as ONE combined portfolio from one CSV or Excel price file per asset - on each weekly or monthly signal day the assets are ranked by past return, the top N are held until the next signal, and the rest are sold (optional absolute-momentum filter to go to cash) - and produces an HTML tearsheet. Use for momentum / relative-strength / rotation tests; not for channel breakout or moving average strategies.
---

# Momentum Backtest (multi-asset rotation)

Runs one fixed strategy: rank all assets by momentum on every signal day, hold the strongest ones, switch at the next signal. All the math lives in `mom_backtest.py` (saved next to this SKILL.md), so every run uses identical rules. Never change the rules, add optimisation, or test other parameter sets unless the user explicitly asks for a separate run.

The user is not a programmer: explain results in simple words, use a real-world analogy when a concept comes up (e.g. momentum = "betting on the runners who are currently leading the race"), and keep replies short.

## Step 1 - Get the price files
- The user gives a folder, or a list of files: one `.csv` / `.xlsx` per asset. At least 2 assets. If none are given, ask.
- Each file MUST contain `date` and `close`, plus `open` unless the user chooses close-only fills (see Step 3, question 10). Case-insensitive; `high`/`low` and other columns are ignored. The date column may also be called `time` (TradingView exports). Dates may be dd/mm/yyyy or yyyy-mm-dd (or native Excel dates); they are never guessed. If anything is missing the script stops and says which file and column - tell the user in plain words. Do not guess or rebuild columns.
- Excel workbooks must have a single sheet each. Asset name = file name, unless the user gives labels (`--names`, same order as the files).
- Files may be daily, weekly or monthly. Daily data works with both weekly and monthly signals. Data coarser than the signal (e.g. monthly files with a weekly signal) is refused by the script.

## Step 2 - Scan, then confirm the start date
1. Run the scan (no backtest yet):
```
py mom_backtest.py --folder "<folder>" --scan
py mom_backtest.py --files "<a.csv>" "<b.csv>" [--names A B] --scan
```
2. Show a small table: each asset's first date, last date, bar count and data frequency, then the **overlapping period** (`overlap_start` to `overlap_end`).
3. Ask: start from the **overlap start date**, or a **different start date**? It must be on or after the overlap start. Dates are dd/mm/yyyy.
4. Tell the user: momentum is measured using any history before the start date when it exists. If there isn't enough history for the lookback, the portfolio simply stays in cash until the first ranking is possible (the script reports that date).

## Step 3 - Ask for the parameters (always ask, never assume)
Ask in this order. Ask 1-2 first, because the lookback unit depends on the answer. Use the multiple-choice question tool if available. Only commission and slippage have defaults.
1. **Signal frequency** - **Weekly** or **Monthly**: how often assets are ranked and the portfolio switched. (Applies to daily data too: daily files with a monthly signal are checked once a month.)
2. **Momentum type** - **Relative** (always hold the top N by return) or **Relative + absolute filter** (hold a top-N asset only if its own return is also positive; otherwise that slot stays in cash).
3. **Lookback** - how far back to measure the return, **in weeks if Weekly, in months if Monthly** (e.g. 6 months).
4. **Skip recent** (optional, default 0) - ignore the most recent N weeks/months of the lookback (a common trick to avoid short-term reversals). Must be smaller than the lookback.
5. **Top N** - how many assets to hold (between 1 and the number of assets).
6. **Starting capital** - for the whole portfolio.
7. **Commission** - flat amount per order. Default 0.
8. **Slippage** - percent of the fill price, against you on every fill. Default 0%.
9. **Rebalancing** - **None** (default: assets that stay in the top N are left alone, so a winner can grow to a big share) or **On each signal day** (`--rebalance on-signal`: assets that stay are trimmed/topped up back to equal weight, equity / N). Explain the trade-off simply: rebalancing keeps the portfolio evenly spread but means many more small orders (and more commission/slippage if costs are set), especially with weekly signals.
10. **Fill price** - only ask if the files have no `open` column (e.g. TradingView exports with just `time,close`): re-export with open prices (recommended, fills at the next bar's open), or **fill at the signal close** (`--fill signal-close`, slightly optimistic because you trade at the same price that produced the signal). If the files have `open`, use the default next-open fill without asking.
11. **Output** - HTML only (default); PDF on request.

The same settings apply to every asset (no per-asset settings).

Fixed rules (do not ask, state them in the confirmation):
- Long only, whole shares only (leftover cash stays in the pool; a buy that cannot afford one share is skipped and flagged).
- **Momentum score** = return over the lookback, measured on each asset's own closes (last close on or before the reference date; with skip: from `t - skip - lookback` to `t - skip`).
- **Signal day** = last trading day of each week / month. Rank on that day's close; orders fill at the **next bar's open** (or, with `--fill signal-close`, at that same close), sells first, then buys. The unfinished final period is never a signal.
- **Selection:** top N by score (ties: file order). Assets that drop out of the top N are sold in full. Assets that stay are left alone, unless rebalancing is on (see below). Each buy uses `cash in pool / number of empty slots` (of N), which gives an equal split.
- **Rebalancing** (if chosen): in the same fill as each signal's sells and buys, every asset that stays held (market open, no pending sell) is brought to `total equity / N` (trims first, then top-ups limited by cash; never trimmed to zero; whole shares). Same price rule, commission and slippage as other orders. Empty slots stay in cash, so it never over-invests.
- **Absolute filter** (if chosen): top-N assets with a score of 0 or below are not held; their slots stay in cash.
- **Calendar:** every date on which AT LEAST ONE asset traded is a portfolio date. On a date when an asset has no bar its market is closed: it cannot be traded (orders wait for its next real bar), and its last close is carried forward for valuation.
- An open position at the end is marked at the last close (no exit cost).
- Benchmark: equal-weight buy-and-hold of ALL assets (capital / number of assets each, bought at the first bar's open with the same costs), rebalanced back to equal weight on every signal day. It is invested from day one, while the momentum portfolio waits in cash until the first ranking is possible.

Repeat the chosen settings back in one short line, then run.

## Step 4 - Run the script
1. Check `pandas`, `numpy`, `matplotlib` and `openpyxl` import; install any that are missing (`pip install <pkg>`).
2. Run (on Windows plain `python` may hit the Microsoft Store stub; use `py`, or the full path to the real Python):
```
py mom_backtest.py --folder "<folder>" | --files <f1> <f2> ... [--names <n1> <n2> ...] \
  [--start dd/mm/yyyy] --frequency weekly|monthly --mode relative|relative-absolute \
  --lookback <N> [--skip <N>] --top-n <N> [--rebalance on-signal] [--fill signal-close] \
  --capital <capital> --commission <commission> --slippage <slippage_pct> [--pdf] --outdir <output folder>
```
   `--slippage` is in percent: `0.05` means 0.05%. Leave `--start` out to use the overlap start. `--lookback` / `--skip` are weeks for `weekly`, months for `monthly`.
3. The script prints JSON. If `"status": "error"`, explain the message in plain words and stop - do not work around it.

## Step 5 - Present the results
- Say where the HTML tearsheet (and PDF if requested), the trade list CSV and the signal history CSV were saved.
- Give 3-5 short lines: CAGR, max drawdown, Sharpe and number of trades for the momentum portfolio vs the equal-weight buy-and-hold, the best and worst asset by contribution, the first ranking date, plus any data warnings. Do not re-list every metric; the tearsheet has them.
- Do not suggest "better" parameters or run extra variations unless asked.

## Tearsheet contents (dark theme)
Strategy rules as selected; momentum portfolio vs equal-weight buy-and-hold (final equity, total return, CAGR, volatility, Sharpe, Sortino, max drawdown, Calmar, longest drawdown); trade statistics; per-asset table (trades, win rate, P&L, contribution, time held, own buy-and-hold); equity curve; drawdown; **holdings-over-time timeline**; number of assets held; monthly returns heatmap; **signal history** (for every signal day: what was selected and the full ranking with scores); price chart with buy/sell markers per asset; combined trade list. Ratios use a zero risk-free rate, annualised from the data frequency. All dates dd/mm/yyyy.

**Trade list:** one row per fill, in date order, tagged `Momentum entry`, `Momentum exit`, `Rebalance buy`, `Rebalance sell` or `Open position (marked at last close)`, with date, asset, signal date, momentum score at the signal (entries/exits), shares, price, commission and value. P&L, return and bars held appear on the exit (or open-position) row and cover the whole position, rebalance top-ups and trims included. Rebalance rows have no P&L of their own.
