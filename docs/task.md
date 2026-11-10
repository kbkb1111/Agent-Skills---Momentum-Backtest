# To-do - Momentum Backtest

## Done
- [x] Clarify requirements with the user (momentum type, frequency order, weighting, location) - 10/11/2026
- [x] Write `mom_backtest.py` (engine, metrics, tearsheet, PDF) - 10/11/2026
- [x] Write `SKILL.md`, README, PRD, implementation plan - 10/11/2026
- [x] Test on synthetic data: scan, weekly, monthly, absolute filter, later start, xlsx, monthly files - 10/11/2026
- [x] Hand-check first ranking, fill price, share count - 10/11/2026
- [x] Test error messages: top N too big, lookback too long, skip >= lookback, start before overlap, tiny capital, one file, missing column, data too coarse - 10/11/2026

- [x] Add close-only fill option for files without `open`; regression + smoke test on 5 XETR weekly ETF files - 10/11/2026

- [x] Run a real backtest on the 5 XETR weekly files (weekly, relative+absolute, 52w, top 2) - 10/11/2026
- [x] Add optional rebalancing on each signal day; regression + hand-check; re-run the test with it on - 10/11/2026

## To do
- [ ] Try with more real price files
- [ ] Decide whether quarterly signals or momentum-weighted sizing are wanted later

## Changelog
- **[10/11/2026]** All tasks above completed; waiting for a test on real data.
