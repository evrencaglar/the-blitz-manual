# How The Blitz Works

A deeper look at the strategy, for users who want to understand what is happening under the hood. Nothing here is required reading to run the EA — the defaults are the intended configuration.

---

## The big picture

The Blitz is a **machine-learning trend/momentum system for Gold**. Two independent engines, Mark 1 (H1) and Mark 2 (H4), each run a pair of gradient-boosted tree ensembles that were trained offline on Gold data from **2018-01-01 to 2024-12-31** and then **frozen**. "Frozen" means the models, thresholds, and exits shipped with the EA are exactly the ones that passed research validation — the EA does not learn, adapt, or re-optimize itself at runtime.

Both engines trade the same symbol in both directions (long and short), each managing at most 2 positions of its own, identified by unique magic numbers (260925 and 260926). The Blitz never touches positions opened by you or by other EAs.

---

## Entries

On every **completed** bar of its timeframe, an engine:

1. **Builds 17 features** from the last 600 bars: RSI-based momentum; price change over 1, 3, 6, 12, and 48 bars; bar range and body relative to volatility; where the close sits inside the bar; current volatility versus its long-run median and versus price; a 48-bar trend-efficiency ratio; hour-of-day and day-of-week cycles (sine/cosine); and the current spread as a cost fraction.
2. **Scores the bar** with both ensembles. The output is a prediction of net profit **in R** (multiples of the amount risked), after typical costs.
3. Takes the better of the long and short scores. If it clears the engine's **edge threshold** — 0.10 R for Mark 1, 0.05 R for Mark 2 — the setup qualifies.

Then the entry gates apply, in order. A trade is skipped when:

- the circuit breaker is pausing that engine (Mark 2 only),
- the engine already has 2 open positions,
- the spread is wider than `MaxSpreadPoints`,
- the bar falls in a blocked server hour (Mark 2: 04, 10, 21, 22),
- a user-configured schedule, time block, or news window is active,
- volatility would force a stop wider than the dollar cap (`Engine1/2_MaxSLDollar`),
- auto sizing cannot fit the trade inside your risk budget even at the broker's minimum volume.

## Exits

Every position always has a real stop loss and take profit on the server side:

| Engine | Stop loss | Take profit | Time stop |
|---|---|---|---|
| Mark 1 (H1) | 3 × ATR | 4 × ATR | 24 bars (~24 h) |
| Mark 2 (H4) | 2 × ATR | 3 × ATR | 6 bars (~24 h) |

ATR is measured on the signal bar, so stops automatically widen in volatile markets and tighten in quiet ones (subject to the dollar caps above).

- **Time stop**: a position that hasn't hit either level within ~24 hours is closed at market. The Blitz is built for quick, decisive moves — dead money is recycled.
- **Trailing stop** (both engines, on by default): once open profit reaches the trigger (1.0× / 0.75× initial risk), the stop starts following price at the trail distance (1.0× / 0.5× initial risk). The stop only ratchets in your favor.
- **Scheduled exits** (optional): end-of-day and Friday close, at broker server time.

## Circuit breaker (Mark 2)

After **3 consecutive losing trades**, Mark 2 pauses new entries for **5 days**, then resumes automatically. The breaker state survives terminal restarts (rebuilt from account history). Mark 1 intentionally has no breaker — in research, its faster trade cadence recovered from losing streaks faster than pausing helped.

## Position sizing

Three mutually compatible layers, all capped by `MaxLots`:

1. **Fixed lots** (default): every trade uses the engine's configured size.
2. **Auto risk sizing** (`UseAutoLotSize`): size computed so a full stop-out loses ~`RiskPercentPerTrade` % of balance. Computed from the actual stop distance and your broker's contract specification. Never rounds up into extra risk — undersized budgets skip the trade.
3. **Dynamic scaling** (optional): streak mode adjusts size after each result (win → bigger, loss → smaller, clamped to `AutoLotMin`/`AutoLotMax`); equity-growth mode scales with the account's high-water mark. Combined with auto sizing, scaling adjusts the *risk %* and remains capped by `MaxEffectiveRiskPercent`.

## Account-level protections (optional)

- **Daily loss limit** and **daily drawdown limit**, in money or % of the day's starting equity. On breach: close Blitz positions only, block entries until the next server day. State is persisted, so a midnight restart doesn't reset the accounting.
- **News filter**: blocks entries around calendar events (configurable currencies, impact, minutes before/after). Live only; fails closed if the calendar is unreadable.
- **Time blocks**: up to four daily no-entry windows.

## The Performance Observer (mock stream)

Live accounts start with zero closed trades, so statistics would be blind for weeks. The Observer fixes this: in the background, it reconstructs a quote-granular mock history of your **exact current configuration** on your broker's recent data (initially 120 days, extending up to 365 if needed for a full 30-close sample), validates M1 data coverage, and caches the result. That powers the MOCK column of the Quant Panel and fills the Performance Bridge until live trades take over.

In the Strategy Tester the Observer is bypassed — tester statistics come from the test's own executed trades.

## What The Blitz does NOT do

- No martingale, no grid, no averaging down — every trade is independent with a hard stop.
- No re-optimization or online learning at runtime.
- No trading during spread spikes, news windows (if enabled), blocked hours, or after daily limits fire.
- No interference with manual trades or other EAs' positions.
