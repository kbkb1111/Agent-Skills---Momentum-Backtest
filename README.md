# Momentum Backtest Skill

A Claude skill that backtests a **momentum rotation strategy** on several assets at once.

**In plain words:** imagine a race where, every week or month, you check who is currently in the lead and put your money on the front-runners. When someone falls behind, you swap them out for a newer leader. This skill replays that idea on your historical price data and shows how it would have done versus simply holding everything.

## What it does
- You upload one price file (CSV or Excel) per asset (at least 2).
- Choose how often to check (weekly / monthly), how far back to measure momentum, and how many assets to hold (top N).
- Optional safety filter: only hold an asset if it is also going up on its own, otherwise sit in cash.
- Output: a dark-theme HTML tearsheet (optional PDF), a trade list CSV and a signal-history CSV.

## Files
| File | Purpose |
|---|---|
| `SKILL.md` | The instructions Claude follows (questions to ask, rules, steps) |
| `mom_backtest.py` | The script that does all the maths and builds the report |
| `docs/PRD.md` | What the skill must do |
| `docs/implementation_plan.md` | How it was built |
| `docs/task.md` | To-do / progress |

## How to use
Ask Claude for a momentum backtest and give it the folder with your price files. Claude will scan the files, ask for the settings, run the script and summarise the results.

Direct command (Windows, use `py` if `python` fails):
```
py mom_backtest.py --folder "<folder>" --frequency monthly --mode relative --lookback 6 --top-n 2 --capital 100000 --outdir "<out>"
```
`--lookback` is in weeks for `weekly`, months for `monthly`.

Files with only `date` and `close` (no `open`) work too: add `--fill signal-close` and trades are made at the signal bar's own close.

Optional `--rebalance on-signal`: on each signal day, assets that stay held are brought back to equal weight (otherwise they are left alone).

## Changelog
- **[10/11/2026]** Added optional rebalancing on each signal day.
- **[10/11/2026]** Added `--fill signal-close` for close-only files.
- **[10/11/2026]** First version: weekly/monthly signals, relative and relative+absolute momentum, equal-weight benchmark, HTML/PDF tearsheet.
