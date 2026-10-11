# FAQ & Troubleshooting

## About the strategy

**What kind of system is it — trend following, mean reversion, scalping?**
The Blitz is a short-term **momentum system driven by a machine-learning model**, trading Gold in both directions. It aims to catch decisive moves early and get out within about a day — the average trade lasts ~18 hours. It is neither a scalper fighting the spread nor a position trader holding for weeks.

**Is it really machine learning, or marketing?**
Really. Every bar is scored by a trained statistical model rather than fixed indicator rules. The important part is how it was validated: the model was trained only on data through the end of 2024, and **2025–2026 was kept as a fully untouched out-of-sample test**. On that unseen data — an unusually erratic Gold market — the backtest shows profit factor 1.69 over 539 trades, matching the full-period 1.68. That consistency is the answer to the overfitting concern every ML-based EA raises.

**Does it use martingale, grid, or averaging down?**
No. Every trade is independent, opens with its own hard stop loss and take profit, and risks a fixed, predefined amount. You can verify this in your own account history within days: one position per signal, never more than 2 open per engine, no recovery sizing.

**What win rate and drawdown should I expect?**
In the 2019–2026 real-tick backtest (fixed 0.01 lots): win rate 60.9 %, profit factor 1.68, maximum equity drawdown 0.54 %. These describe strategy quality at minimum size — your absolute numbers scale with your lot size. Losing streaks still happen: runs of 8+ losses occurred historically in about 4.8 % of cases, and the longest stagnation lasted 291 days. Size your account so a bad month is uncomfortable, never fatal.

**How will I know if it stops working?**
That's exactly what the Quant Panel is for: it compares your live results against the strategy's historical reference profile (expectancy, profit-factor drift, stagnation, losing streaks, and more) and flags when performance leaves the normal envelope. You get an early warning before it shows in your balance. See [dashboard.md](dashboard.md).

## Getting started

**Which symbol and timeframe should I use?**
Gold (your broker's XAUUSD symbol) with the chart on **H1** — that is the default, tested configuration. The two models read their own separate timeframes internally, so attaching to a different chart timeframe does not break anything, but H1 is recommended.

**Which account type do I need?**
A **hedging** MetaTrader 5 account. On netting accounts the EA prints "The Blitz requires an MT5 hedging account" and refuses to start. If you are unsure which type you have, ask your broker — the account specification states it, and many brokers can convert or open a hedging sub-account.

**Does it work on other symbols (EURUSD, indices, crypto)?**
The models were trained exclusively on Gold. The EA will technically run on other symbols, but the strategy's edge is validated for Gold only. Not recommended.

**What account size do I need?**
There is no hard minimum — with the default 0.01 lots, the binding constraint is your broker's margin requirement for Gold. Check the margin shown when you place a 0.01-lot trade manually, and keep comfortable headroom for 4 simultaneous positions (2 per engine) plus drawdowns.

**Do I need a VPS?**
Not required, but strongly recommended. The Blitz manages stops on the broker server, so your trades are protected even if your PC is offline — but time stops, trailing, daily limits, and scheduled exits only run while the terminal is up. A VPS keeps those active 24/5.

## "Why is it not trading?"

This is the most common question, and the answer is almost always one of these, roughly in order:

1. **The model sees no qualifying setup.** The Blitz is selective by design; quiet stretches of a day or more are normal, especially for Mach 2, which is the slower model and additionally has a session filter. This is the usual reason.
2. **The Mach card says PAUSED.** Mach 2's circuit breaker triggered after 3 consecutive losses and is resting for 5 days. It resumes by itself.
3. **The spread gate.** If your broker's Gold spread regularly exceeds 50 points, entries are skipped. Either trade during liquid hours or raise `MaxSpreadPoints` cautiously.
4. **News filter active.** Check the footer: `NEWS WINDOW / entries paused` or `Calendar unavailable / entries paused` both block entries.
5. **Time blocks / schedules.** A daily block window or an end-of-day close you configured is active.
6. **Volatility cap.** In extreme volatility the stop would exceed `Engine_MaxSLDollar`, so the trade is skipped. This protects you from the most expensive stops.
7. **Risk budget too small.** With auto sizing on, if even the broker's minimum lot would risk more than your configured %, the trade is skipped rather than oversized. Lower risk expectations or raise the account balance.
8. **Algo Trading off.** The toolbar button must be green, and the EA's Common tab must allow algorithmic trading.

## Backtesting

**How do I backtest it?**
Open the Strategy Tester (Ctrl+R), select The Blitz, symbol XAUUSD, **H1** timeframe, "Every tick based on real ticks" for the most faithful modeling. Both engines run inside the tester.

**What behaves differently in the tester?**
- The news filter is automatically bypassed (MT5 has no calendar in the tester).
- The Performance Observer mock is off; tester statistics come from the test's own executed trades.
- Panels update in visual mode; in non-visual tests the trading logic runs at full speed.

**Backtest looks different from the dashboard's MOCK column?**
They measure different things: the mock replays recent broker history with your live settings, while a backtest covers whatever period you chose, with tester-specific execution modeling. Small differences in spread, fills, and history are expected.

## Risk and money management

**What risk per trade should I use?**
The default fixed 0.01 lots is the conservative baseline. If you switch to auto sizing, 0.5–1.0 % per trade is a sane retail range; remember both engines can hold 2 positions each, so worst-case open exposure is roughly 4 × your per-trade risk (plus trailing effects).

**What is the worst losing streak I should expect?**
Research reference: completed runs of **8 or more consecutive losses occurred in about 4.8 % of cases**, and the longest observed stagnation (time below the last equity high) was **291 days**. The Quant Panel tracks both metrics live so you always know where you stand relative to history. Size your account so that such a streak is uncomfortable, not fatal.

**Is it prop-firm friendly?**
The tools are there: enable `EnableDailyLimits` with `DailyLossLimit` set below your firm's daily loss rule, and use `MaxLots` / risk % to cap exposure. Verify the firm's exact rule definitions (equity vs. balance, server day boundaries) and leave a safety buffer — slippage can push a realized loss slightly past any software limit.

**Does it use martingale or grids?**
No. Every trade is independent, with a predefined stop loss, take profit, and maximum lifetime.

## Operation

**What happens if the terminal restarts or the power goes out?**
Server-side stops and targets keep protecting open positions. On restart, the EA restores its state: daily-limit accounting, circuit-breaker status, streak sizing, live results ledger, and panel snapshots are all persisted and rebuilt from broker history.

**Can I run other EAs or trade manually on the same account?**
Yes. The Blitz only manages positions carrying its own magic numbers (260925 / 260926). Daily limits, however, measure **account equity** — other trading affects them.

**Can I run it on several charts or accounts?**
One chart per account/symbol is the intended setup. State files are keyed per account + server + symbol, so different accounts don't clash. Do not attach two copies to two Gold charts on the same account — they would share magic numbers and double the intended exposure.

**Which time zone do the schedules use?**
Everything time-based uses **broker server time**, shown in the dashboard footer (`Broker time …`). Check that line before configuring time blocks or scheduled exits.

**How do I update to a new version?**
Close the chart's EA (or the terminal), replace the `.ex5` in `MQL5/Experts`, reattach. Persisted state is keyed per account and symbol and carries over. When an update changes persisted formats, the release notes will say so.

**Where are the log and data files?**
`File → Open Data Folder → MQL5/Files`: the live-results ledger, dashboard snapshot, mock cache, and (with `LogDeals` on) `Blitz_deals.csv` with every deal's full cost breakdown. Terminal journal entries from the EA appear in the Experts log.

## The panels

**The MOCK column says "N/A" everywhere.**
The Observer is still bootstrapping (watch the progress bar), or your broker's history for the symbol is too short/gappy to build 30 mock closes. The panel reports this honestly rather than estimating from thin data. It fills in automatically as coverage improves.

**A statistic turned red. Should I stop the EA?**
Single red cells are part of normal variance — that's why the panels exist: to show you the *distribution*, not just the average. The Performance Bridge's `CAUTION / resembles stagnation periods` verdict is the aggregated warning. If several statistics degrade together over weeks and the caution verdict persists, review position size first rather than switching the system off mid-drawdown — the research references (21-day median stagnation, 291-day worst case) exist precisely so you can judge whether what you see is still inside the expected envelope.

**Can I hide the panels?**
Yes: `ShowDashboard` and `ShowQuantPanel` can each be switched off, and `DashX`/`DashY`, scale, and font settings adjust placement and readability.
