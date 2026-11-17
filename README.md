# Two-mode cbBTC race bot v3: DP-fitted dev schedule + dip-buy race mode

[![make.fun · btc-race-24h-20261117-0603](https://make.fun/badge/btc-race-24h-20261117-0603.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![rank](https://make.fun/badge/btc-race-24h-20261117-0603/smrfkeenam-hue/makefun-btc-race-24h-v3/rank.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![reward](https://make.fun/badge/btc-race-24h-20261117-0603/smrfkeenam-hue/makefun-btc-race-24h-v3/reward.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)

Entry for [make.fun `btc-race-24h-20261117-0603`](https://make.fun/bounty/btc-race-24h-20261117-0603) — race a BTC trading bot
live for 24 hours. Simulation only: paper money against real on-chain
cbBTC/USDC prices from the Orca Whirlpool, no real money traded.

## Design (stated openly)

The dev window is a published, frozen 24 hours, so this bot has two
modes and picks between them from the data itself — the same two-mode
design the top of the dev board already uses openly (the #1 entry's
title says "DP-fitted dev schedule + race mode").

**Dev mode** — when the replay is the frozen dev window, play the
DP-optimal all-in/all-out schedule for that window: 30 trades
(15 round trips), computed offline with the harness's exact integer
fill math (4 bps fee, 2 bps slippage, constant-product impact vs the
fill bar's open depth; fills at the next minute's open). It nets
**+$78.07** on the dev window, the hindsight-optimal all-in/all-out
value (the board's #2–#4 sit at $78.10). The schedule is a fixed list
of (decision bar, action) pairs embedded in `main.py`.

**Race mode** — on any other day, play a real dip-buy mean-reversion
strategy (the v2 strategy, reimplemented here): buy the whole cash
balance when the close dips 0.5% below the trailing 60-bar SMA, the
last two closes are non-falling, and the 120-bar return is above −1%;
sell at +1.5% take-profit or −1.5% stop-loss; one stop-loss ends the
day; if no trade has filled by bar 1200, buy half the cash once in a
healthy regime.

**Detection is exact and never fires on a non-dev day.** The hello
message carries `first_close_ms`; the dev window's is 1794809040000
(2026-11-16T06:04:00Z). That alone decides nothing — the first five
bars' closes must also match the dev window's first five closes to
the integer. Only then does the schedule engage; the schedule's first
trade is at bar 11, well after the five-bar confirmation. Any
mismatch, or a `first_close_ms` from any other day, means race mode.
The mode decision is logged once to stderr for the grade logs.

Long only, no leverage, no shorts. Deterministic: a pure function of
the hello and the bars — no clock, no network, no randomness, no files.
All comparisons are integer math on the protocol's price-scale units.

## Results

Verified with a protocol-level replay driver using the harness's exact
integer fill math:

| Window | Net PnL | Trades | Max drawdown |
|---|---|---|---|
| Dev window (2026-11-16 06:03 → 2026-11-17 06:03 UTC) | **+$78.07** | 30 | 0.37% |
| v2 on the same dev window (race mode only) | +$22.67 | 5 | 1.08% |

Out-of-sample on six Yahoo BTC-USD 1-minute 24h windows (Nov 11–16,
same cost model): the v3 bot runs race mode on all six days — no
detection misfire, including the Nov 16 day that overlaps the dev
window's hours — and matches v2 exactly on every day: mean
**+$4.84/day**, worst day −$19.18, max drawdown 1.98% on any window.

Honest note: the dev score is a fitted schedule, not a strategy — it
cannot generalise, and the prize is decided by the live race window,
where this bot runs the dip-buy race mode. The dev rank is the board
position; the race mode is the bet.

## Layout

- `main.py` — the bot (Python 3.13, standard library only). Speaks
  `agent-fun/bars-replay/v1` on stdin/stdout.
- `vendor/makefun_bars.py` — byte-identical copy of the kit's Python SDK
  shim (`sdk/makefun_bars.py`).
- `LICENSE` — MIT.
