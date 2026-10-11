# Dashboard & Quant Panel Guide

The Blitz shows two panels on the chart. The **dashboard** (left) answers *"what is the EA doing right now?"* The **Quant Panel** (right) answers *"is the strategy behaving the way its research history says it should?"*

> Terminology: the two models are called **Mach 1** and **Mach 2** throughout this manual; the on-chart panels label their cards **MARK 1** and **MARK 2**. Same models, two labels.

![The Blitz dashboard and Quant Panel on a live chart](images/dashboard-panels.png)

*A real live view of both panels. This snapshot happens to show a CAUTION state — the Performance Bridge flagged that recent results resembled historical stagnation periods, and Market Health read drought-like conditions. Both engines stayed ACTIVE and kept trading normally: the panels inform, you decide. Every field visible here is explained below.*

Both panels refresh on a 2-second timer. Statistics based on closed trades recompute after every closed position and at startup.

---

## Part 1 — The Blitz dashboard

### Header

`THE BLITZ` with the version number. Nothing interactive — just confirmation of which build is running.

### MARK 1 / MARK 2 cards

One card per engine.

| Field | Meaning |
|---|---|
| **State** | `ACTIVE` (green) — trading normally. `PAUSED` (yellow) — the circuit breaker is resting this engine after consecutive losses; entries resume automatically when the pause expires. `OFF` (grey) — the engine is disabled in settings. |
| **N open** | How many positions this engine currently has open (maximum 2 per engine). |
| **Size line** | `0.01 lots` in fixed mode, `Risk 1.00%` when auto risk sizing is on, or `Scaled size` when dynamic scaling is active. |

### MARKET HEALTH

A structural read of the **Gold daily chart**, computed only from completed daily bars. It is **informational only — it never blocks or triggers trades**. Think of it as a weather report for the kind of trend environment the strategy was built for.

| Row | What it measures | Reading it |
|---|---|---|
| **BULL FAN** (x / 3) | How many of the three bullish alignments are in place: price above the 20-day average, 20-day above 50-day, 50-day above 200-day. | 3 = fully stacked bullish structure. Green when above 1, red at 1 or below. |
| **DISTANCE ABOVE SMA200** (+x.x ATR) | How far the last daily close sits above the 200-day average, measured in ATRs (daily volatility units). | Green at +1.0 ATR or more; red below. Deeply negative = strong downtrend. |
| **FULL BULL DAYS / 10** | Of the last 10 completed days, how many had the full fan *and* a rising 50-day average. | Green when at least 1; red at zero (no clean bullish day in two weeks). |
| **SMA50 slope / Efficiency** | Slope of the 50-day average in ATRs, and the efficiency ratio (how directly price travelled over the last 10 days: 1.0 = perfectly straight move, near 0 = choppy). | Neutral context lines — no color coding. |
| **Net 2-week move** | Net change over the last 10 completed days, in ATRs. | Neutral context. |

**Verdict line** — the three colored rows vote:

- `CLEAR / historical earning conditions` (green) — all three votes positive: the market structure resembles periods where the strategy historically earned.
- `NEUTRAL / mixed structure` (yellow) — two of three.
- `CAUTION / drought-like conditions` (red) — one or none: structure resembles historical dry spells.
- `WAITING FOR DAILY HISTORY` (grey) — not enough completed daily bars on this chart yet; the EA needs 220.

### PERFORMANCE BRIDGE

Your strategy's recent track record, tracked over the **most recent 30 closed results**. Until your account has enough live trades, the window is filled by the **mock stream** (a historical simulation of your exact configuration — see the Observer settings); live results replace mock results one by one as they arrive. The window header shows the mix, e.g. `18 MOCK / 12 LIVE`, or `TESTER / N COMPLETED RESULTS` inside the Strategy Tester.

| Metric | Meaning | Red flag |
|---|---|---|
| **PROFIT FACTOR** | Gross wins ÷ gross losses over the window. 1.50 means $1.50 won for every $1 lost. | Red below 1.10. |
| **AVERAGE R** | Average result per trade in R (multiples of the money risked). Positive = the average trade earns. | Red when negative. `N/A` if the window can't be risk-normalized yet. |
| **LOSS RUN** | Worst losing streak in the window (consecutive losses). | Red at 8 or more. |

Values show `--` until at least 5 results exist.

**Verdict line** underneath:

- `CAUTION / resembles stagnation periods` (red) — statistics match the worst historical periods (loss run ≥ 8, or profit factor < 1.10 with negative average R). The EA keeps trading; this is a signal to *you* to pay attention, not an automatic shutdown.
- `Indicative bridge / live replacing mock` (green) — window still contains mock results.
- `Live account window` (green) — all 30 results are real trades.
- `Waiting for 30-result bridge` (grey) — still collecting results.

The thin bar under the metrics is the mock bootstrap progress (green when complete, red on failure).

### Footer

| Line | Meaning |
|---|---|
| **Day … / peak DD …** | Today's equity change and the give-back from today's equity peak (only when daily limits are enabled). Turns red with `DAILY LIMIT HIT / closing Blitz trades` when a limit fires. `Daily limits off` otherwise. |
| **News line** | `News filter off`, the next relevant event (`Next news USD 2026.10.14 14:30`), `NEWS WINDOW / entries paused` while inside a blocked window, or `Calendar unavailable / entries paused` if the calendar can't be read. `News filter unavailable in tester` in backtests. |
| **Broker time** | The broker server clock. All scheduled exits and time blocks use this time — check it before setting them. |

---

## Part 2 — The Quant Panel

The Quant Panel compares two data streams side by side:

- **MOCK** — the Observer's historical simulation: your *current settings* replayed over your broker's recent data (up to ~1 year). Header `MOCK n/30` shows how many of the 30 window slots it fills. `MOCK OFF` in the Strategy Tester.
- **LIVE** — your account's real executed Blitz results. Header `LIVE n/30` (or `TESTER n/30`).

`N/A` means the metric honestly refuses to guess — the underlying sample is too small. That is by design: the panel never claims a complete statistic from an incomplete sample.

### Row by row

**1. EXPECTANCY / 95% CI** — `+0.42 +/- 0.31 R`
Average result per trade with a 95 % confidence interval (t-interval) around it, computed over each stream's own 30-close window. If the interval spans zero, the edge is not statistically distinguishable from luck yet.

**2. PF30 DRIFT / ROBUST SIGMA** — `-1.3 sigma`
How far the current 30-trade profit factor sits from the frozen research median (1.526), measured in robust standard deviations (IQR 1.669). Green above −1σ, yellow between −1σ and −2σ, red below −2σ. This is the early-warning gauge: profit factors wobble naturally, and this row tells you whether the wobble is still normal.

**3. SINCE REALIZED HIGH** — `34 days`
Days since the equity curve of that stream last made a new high (stagnation length). Research references: median 21 days, 90th percentile 58 days, worst case 291 days. Yellow beyond 58 days. Long stagnations are a normal part of this strategy's profile — this row tells you where you stand in that distribution.

**4. LOSS-RUN SURVIVAL** — `L5 / 82th pct`
The current or worst recent losing streak (L5 = five losses in a row) and which percentile of historical losing streaks it equals. In research, completed runs of 8+ losses occurred in 4.8 % of cases. Red at L8 or more.

**5. KELLY / RISK SNAPSHOT** — `0.62x Kelly` or `12.4% estimate`
The Kelly criterion — the theoretically growth-optimal risk fraction — estimated from the sample. Live also shows your *current open risk* as a multiple of that optimum when it can be computed (`0.62x Kelly` = risking 62 % of the Kelly amount, i.e. conservative). `0% estimate` means the sample shows no positive edge. Above ~1x Kelly you are risking more than the growth-optimal level.

**6. EQUITY CURVE QUALITY** — `S 1.85 / R2 0.92`
Sharpe-style ratio (S) and R² of a straight line fitted to the last 30 completed sessions' equity. R² near 1 means a smooth, consistent climb; low R² means a jagged path even if profitable.

**7. DAILY VaR / 95 %** — `1.85% loss`
Value-at-Risk: on the worst 5 % of days (of the last 60 completed sessions), the stream lost at least this much. If you set a daily loss limit, the row compares against it and turns **red** when typical bad days would breach your limit — a hint to loosen the limit or reduce size.

**8. MARK DIVERSIFICATION** — `corr +0.21`
Correlation between Mach 1 and Mach 2 daily results over 60 completed sessions. Low or negative correlation means the two models smooth each other — that is the point of running both.

**9. VOLATILITY REGIME / SHARED D1** — `74th pct`
Where today's volatility (ATR14 on daily bars) ranks against the last 504 trading days. One shared value for both columns, since both engines trade the same market. High percentiles = unusually volatile conditions (wider stops, larger swings).

**10. LIVE − MOCK DIVERGENCE** — `+0.35 R gap`
Compares live results against mock results over the same closed-day horizon (needs at least 10 valid results in each stream). Green when live is ahead of the mock, red when behind. Persistent large negative gaps usually mean execution costs (spread, slippage, commission) exceed the mock's assumptions — check `ObserverCostPerLot`.

### Quant Panel footer

Status of the data feeds: `TICK TIME COVERAGE 96.3% / 1,240,551 quotes / 2026.10.10 21:00` while the mock bootstraps, `SAVED LIVE SNAPSHOT / BROKER HISTORY SYNC` while restoring state after a restart, or `QUOTE-TICK MOCK / EXECUTED ACCOUNT / BRIDGE ON LEFT` in steady state.

---

## A practical reading routine

1. **Glance at the Mach cards** — both ACTIVE? Good.
2. **Check the footer** — any limit hit or news block explains quiet periods instantly.
3. **Weekly, read the Quant Panel LIVE column top to bottom** — expectancy interval still positive? PF30 drift within ±2σ? Loss run under 8? Then the strategy is inside its historical envelope.
4. **Treat single red cells as information, not alarms.** The pattern that matters is several rows degrading together over weeks — that is what the CAUTION verdict on the Performance Bridge aggregates for you.
