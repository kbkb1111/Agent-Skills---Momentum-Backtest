# PRD - Momentum Backtest

## Goal
Let the user upload price files for several assets, apply momentum rules to choose which assets to hold until the next momentum signal, and read the result in a clear tearsheet.

## Requirements
- Input: one CSV/Excel price file per asset (`date`, `close`, plus `open` unless close-only fills are chosen), at least 2 assets. Date column may be named `time`; dates dd/mm/yyyy or yyyy-mm-dd. Files may be daily, weekly or monthly.
- Question order: signal frequency first (Weekly / Monthly), then momentum type, then lookback **in that frequency's unit** (weeks or months), optional skip-recent, top N, capital, commission, slippage.
- Momentum type chosen each run: Relative (top N by return) or Relative + absolute filter (own return must be > 0, else slot stays in cash).
- Score = return over the lookback on each asset's own closes; history before the start date is used when it exists.
- Signal on the last trading day of the week/month; fill at the next bar's open; sells before buys; equal split of cash across empty slots; assets that stay are not trimmed.
- Commission and slippage on every fill. Whole shares.
- Date handling: overlap period shown, user may pick a later start; every date any asset traded is kept, closed markets cannot trade that day, last close carried forward.
- Data coarser than the signal frequency is refused with a plain-language error.
- Output: dark HTML tearsheet (optional PDF), trade list CSV, signal history CSV. Dates dd/mm/yyyy.
- Benchmark: equal-weight buy-and-hold of all assets, rebalanced on every signal day.

- Optional rebalancing (`--rebalance on-signal`, default off): on each signal day assets that stay held go back to equity / top-N; shown in the trade list as Rebalance buy/sell.

## Out of scope
Momentum-weighted position sizes, short selling, parameter optimisation, per-asset settings, quarterly signals, volatility targeting.

## Changelog
- **[10/11/2026]** Added optional rebalancing on each signal day (user request: the first version never trimmed or topped up assets that stayed in the top N).
- **[10/11/2026]** First version of requirements, agreed with the user (relative / relative+absolute chosen per run, equal weight, location with the other skills).
- **[10/11/2026]** Added optional close-only fills (`--fill signal-close`) so files with only date+close (e.g. TradingView weekly exports) work; trades fill at the signal bar's own close. Default stays next-bar open. Requested by user for their XETR weekly files.
- **[10/11/2026]** Added: frequency asked first, lookback in matching unit (weeks/months); quarterly dropped; daily data supported with weekly/monthly signals.
