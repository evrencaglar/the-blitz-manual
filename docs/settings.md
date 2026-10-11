# Settings Reference

Every input parameter of The Blitz v1.06, grouped exactly as in the EA's settings dialog. Defaults are the shipped values, tuned for **Gold (XAUUSD)** with the chart on **H1**.

> Rule of thumb: the strategy itself is frozen. These settings control **how much** you trade and **when the EA steps aside** — not the model's decisions.

---

## Marks

| Setting | Default | What it does |
|---|---|---|
| `Engine1_H1_Enabled` | `true` | Switches Mach 1 on or off. When off, it opens no new trades; existing ones are still managed to exit. |
| `Engine2_H4_Enabled` | `true` | Same for Mach 2. |
| `Engine1_Lots` | `0.01` | Fixed lot size for every Mach 1 trade. Ignored when auto sizing or dynamic scaling provides the size instead. |
| `Engine2_Lots` | `0.01` | Fixed lot size for every Mach 2 trade. |
| `MaxSpreadPoints` | `50` | Entry filter: if the current spread is wider than this (in points), the entry is skipped. Protects you from entering during thin or chaotic pricing. |

---

## Risk management

| Setting | Default | What it does |
|---|---|---|
| `UseAutoLotSize` | `false` | `false` = fixed lots (the two settings above). `true` = the EA computes each position size so that a full stop-out loses about `RiskPercentPerTrade` % of the account **balance**. |
| `RiskPercentPerTrade` | `1.0` | Base risk per trade in % of balance, used only when `UseAutoLotSize` is on. |
| `MaxEffectiveRiskPercent` | `2.0` | Hard ceiling on the risk % after any dynamic-scaling boost. Even if scaling doubles your base risk, it can never exceed this. |
| `MaxLots` | `1.0` | Absolute lot ceiling, regardless of sizing method. No Blitz position will ever be larger than this. |
| `Engine1_MaxSLDollar` | `50.0` | Mach 1 entry gate: if the required stop distance is wider than $50 per ounce (high volatility), the trade is skipped. `0` disables the gate. |
| `Engine2_MaxSLDollar` | `80.0` | Same gate for Mach 2 ($80/oz). |

**How auto sizing works (plain version):** the EA measures the stop distance, converts it to money using your broker's contract specification, and picks the lot size that puts your chosen % at risk. It **never rounds up** to the broker's minimum volume — if even the minimum lot would risk more than your budget, the trade is skipped entirely.

---

## Dynamic scaling (optional)

Off by default. Lets the trade size react to recent results or account growth instead of staying fixed.

| Setting | Default | What it does |
|---|---|---|
| `EnableDynamicScaling` | `false` | Master switch for scaling. |
| `AutoLotMode` | `AL_OFF` | `AL_STREAK` = adjust size after each closed trade based on win/loss streaks. `AL_EQUITY_GROWTH` = scale size with account growth (current equity high-water mark ÷ equity at EA start). |
| `AutoLotStart` | `0.01` | Starting lot when scaling begins. |
| `AutoLotMin` | `0.01` | Smallest lot scaling may reach. |
| `AutoLotMax` | `0.05` | Largest lot scaling may reach (safety cap, applied on top of `MaxLots`). |
| `WinAdjustMode` | `ADJ_MULTIPLIER` | After a **win**: multiply the current lot by `StreakWinMult`, or add `StreakWinAdd` lots when set to `ADJ_FIXED_LOT`. |
| `StreakWinMult` | `1.5` | Win multiplier (e.g. 0.02 lots → 0.03 after a win). |
| `StreakWinAdd` | `0.01` | Win increment in fixed-lot mode. |
| `LossAdjustMode` | `ADJ_MULTIPLIER` | After a **loss**: multiply by `StreakLossMult`, or subtract `StreakLossSub` in fixed-lot mode. |
| `StreakLossMult` | `0.7` | Loss multiplier (shrinks size after losses). |
| `StreakLossSub` | `0.01` | Loss decrement in fixed-lot mode. |

Each engine keeps its **own** streak size, and the current value is remembered across terminal restarts. When combined with `UseAutoLotSize`, the streak/equity factor scales the *risk percentage* instead of the raw lot — and `MaxEffectiveRiskPercent` still caps the result.

---

## Daily limits (optional)

Off by default. Emergency brakes measured on **account equity**, per server day. When a limit is hit, the EA **closes only its own Blitz positions** and blocks new entries until the next server day. Your other positions are never touched.

| Setting | Default | What it does |
|---|---|---|
| `EnableDailyLimits` | `false` | Master switch. |
| `LimitUnit` | `LIMIT_MONEY` | Whether the two limits below are amounts in account currency or percentages. `LIMIT_PERCENT` = % of the day's starting equity. |
| `DailyLossLimit` | `0.0` | Maximum loss for the day (equity vs. day start). `0` = off. |
| `DailyDrawdownLimit` | `0.0` | Maximum give-back from the day's equity **peak**. `0` = off. |

The dashboard footer shows live progress: `Day +12.40 USD / peak DD 3.10 USD`, and turns red with `DAILY LIMIT HIT / closing Blitz trades` when triggered.

---

## News filter (optional, live trading only)

Off by default. Blocks **new entries** around economic-calendar events. Open trades are managed normally.

| Setting | Default | What it does |
|---|---|---|
| `NewsFilterEnabled` | `false` | Master switch. |
| `NewsCurrencies` | `"USD"` | Which event currencies to watch, comma-separated (e.g. `USD,EUR`). For Gold, USD is the essential one. |
| `NewsMinutesBefore` | `30` | Block entries this many minutes before an event. |
| `NewsMinutesAfter` | `15` | Keep blocking this many minutes after the event. |
| `NewsImpactLevel` | `IMPACT_HIGH` | Minimum event importance: high only, medium and above, or all events. |

Important behaviors:

- Uses the **built-in MT5 economic calendar** — no external websites to whitelist.
- **Live trading only.** The Strategy Tester has no native calendar, so the filter is automatically bypassed in backtests.
- **Fails closed.** If the filter is on but calendar data is unavailable, entries pause and the dashboard shows `Calendar unavailable / entries paused` — the EA would rather miss trades than trade blind into news you asked to avoid.

---

## Scheduled exits (server time)

Off by default. All times are **broker server time**, not your local PC time — the dashboard footer always shows the broker clock so you can align them.

| Setting | Default | What it does |
|---|---|---|
| `CloseEndOfDay` | `false` | Close all Blitz positions every day at `EODCloseTime`. |
| `EODCloseTime` | `"23:55"` | Time of the daily close, `HH:MM`. |
| `CloseOnFriday` | `false` | Close all Blitz positions on Friday at `FridayCloseTime` (avoids weekend gaps). |
| `FridayCloseTime` | `"22:00"` | Time of the Friday close, `HH:MM`. |

---

## Daily time blocks: no new entries (server time)

Off by default. Up to four independent windows per day during which **no new entries** are taken (open trades are still managed). Each block has an on/off switch plus start and end times in `HH:MM`.

| Setting | Default |
|---|---|
| `Block1On` / `Block1Start` / `Block1End` | `false` / `"12:00"` / `"14:00"` |
| `Block2On` … `Block4On` (with start/end) | `false` / `"00:00"` / `"00:00"` |

Example: block 12:00–14:00 to skip the thin lunch hours, or block the rollover hour if your broker's spreads widen then.

---

## Mach 1 trailing

| Setting | Default | What it does |
|---|---|---|
| `Engine1_UseTrailing` | `true` | Enables the trailing stop for Mach 1. |
| `Engine1_TrailTriggerMult` | `1.0` | Activation: once price has moved in your favor by this multiple of the **initial** stop distance, trailing starts. 1.0 = after +1 R of open profit. |
| `Engine1_TrailDistMult` | `1.0` | Trail distance: the stop follows this multiple of the initial stop distance behind the current price. The stop only ever moves in your favor, never backwards. |

## Mach 2 trailing

Same meaning for Mach 2. Defaults are tighter because Mach 2's winners are shorter-lived: trigger `0.75`, distance `0.5`.

| Setting | Default |
|---|---|
| `Engine2_UseTrailing` | `true` |
| `Engine2_TrailTriggerMult` | `0.75` |
| `Engine2_TrailDistMult` | `0.5` |

---

## Dashboard

| Setting | Default | What it does |
|---|---|---|
| `ShowQuantPanel` | `true` | Shows the separate Quant Panel (statistics monitor). |
| `ShowDashboard` | `true` | Shows the main Blitz dashboard. |
| `DashX` / `DashY` | `16` / `32` | Panel position on the chart, in pixels from the top-left corner. |
| `DashScalePercent` | `80` | Size of both panels, 60–120 %. 100 = original design size. |
| `DashFontScalePercent` | `100` | Text size, 60–140 %, independent of panel size. |
| `DashMinFontSize` | `9` | Smallest allowed text size in points (6–12) so labels never become unreadable. |

---

## Performance Observer: broker quote-tick mock stream

The Observer replays the **exact current configuration** over your broker's recent historical data, so the MOCK column of the Quant Panel is populated from day one — before your account has 30 real closed trades.

| Setting | Default | What it does |
|---|---|---|
| `ObserverMockBootstrap` | `true` | On live charts, build the mock history in the background. The Strategy Tester always bypasses this and uses its own executed trades. |
| `ObserverInitialDays` | `120` | Initial lookback in calendar days. Automatically extended (up to the max below) if 120 days don't yield 30 mock closes. |
| `ObserverMaxDays` | `365` | Maximum lookback. If even that yields fewer than 30 closes, the panel honestly reports "Fewer than 30 mock closes in lookback" instead of inventing statistics. |
| `ObserverChunksPerTimer` | `8` | Background processing speed: cached hourly chunks per timer tick. Higher = faster bootstrap, more CPU. |
| `ObserverCostPerLot` | `0.0` | Estimated round-trip commission + swap per lot, in account currency, deducted in the mock. Set this to your broker's real Gold commission for realistic mock statistics. |

The mock is cached per account/server/symbol, validates broker M1 data coverage, and shows its progress on the dashboard (`TICK TIME COVERAGE`).

---

## Diagnostics

| Setting | Default | What it does |
|---|---|---|
| `LogFeatureParity` | `false` | Writes `Blitz_H1_parity.csv` / `Blitz_H4_parity.csv` (per-bar feature values and model scores) to the terminal's Files folder. For developers verifying live/tester parity — most users can ignore it. |
| `LogDeals` | `true` | Appends every Blitz deal to `Blitz_deals.csv` with position IDs, engine tag, volume, price, profit, swap, commission, and fee. Useful for bookkeeping and support requests. |
