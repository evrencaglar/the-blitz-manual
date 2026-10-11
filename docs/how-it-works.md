# How The Blitz Works — General Principles

This page explains *what kind of system* The Blitz is and how it protects your account. It deliberately stays at the level of principles — the model internals are proprietary — while everything that affects your money is described openly.

---

## The strategy in brief

The Blitz is a **machine-learning momentum system for Gold**:

- **Machine learning, not hand-coded rules.** A statistical model — trained on years of historical Gold data — scores every completed bar and decides whether a trade is worth taking. It replaces the rigid "if indicator X crosses Y" logic of classic EAs with a learned estimate of expected return.
- **Short-term momentum, both directions.** The Blitz trades long and short, aiming to capture decisive moves as they start. It is not a position trader and not a buy-and-hold system: the average trade lasts about 18 hours, and no trade is allowed to live longer than roughly a day.
- **Two complementary engines.** Two instances of the strategy run side by side on different market rhythms (hourly and 4-hour). Each manages its own trades independently, and because they don't always move in lockstep, running both smooths the overall equity curve. The dashboard even shows you their live correlation.
- **Selective by design.** Most bars produce no trade. Quiet days are a feature, not a malfunction — the model only engages when the expected edge clears its bar.

## Machine learning without the usual catch

The standard failure mode of ML trading systems is overfitting: the model memorizes history and collapses on new data. The Blitz was developed specifically to withstand that:

- The model was trained exclusively on data **up to the end of 2024**.
- **All of 2025 and 2026 was reserved as a strictly untouched out-of-sample period** — never used for training, tuning, or selection.
- That out-of-sample window turned out to be an unusually harsh test: the erratic, volatile Gold behavior of 2025–2026. The strategy's out-of-sample statistics (profit factor 1.69 over 539 trades) match its full-period profile (1.68) — the signature of a model that generalized rather than memorized.
- The model is **frozen** at release. The EA does not re-train or quietly change its behavior at runtime; what you test is what you run.

## How every trade is protected

Each trade is independent — **no martingale, no grid, no averaging down, no adding to losers** — and always carries:

- **A hard stop loss and take profit**, placed with the order from the first second. Both adapt to current volatility, so risk scales with the market instead of fighting it.
- **A time stop.** Trades that don't resolve within about a day are closed at market. Capital is recycled rather than parked in dead positions.
- **A trailing stop.** Once a trade moves far enough into profit, the stop follows price and locks in gains. It only ever moves in your favor.
- **Entry quality gates.** Entries are skipped when the spread is abnormally wide or when volatility would force an unreasonably expensive stop.
- **A circuit breaker** on the slower engine: after a cluster of consecutive losses, that engine pauses itself for several days, then resumes automatically.

## Advanced order management

Opening a trade is only the beginning. The Blitz runs an active order-management layer over every live position:

- **Server-side protection from second one.** Stop loss and take profit are placed with the order itself, resting on the broker's server — open positions stay protected even if your PC or internet goes down.
- **Live stop management.** The trailing stop continuously re-prices as the market moves in your favor, respecting the broker's minimum stop distance and never moving backwards.
- **Real-execution handling.** Spread and slippage checks before entry, automatic adaptation to each broker's filling mode, volume steps, and minimum volumes; partial fills are accepted; rejections and failed modifications are logged visibly rather than ignored.
- **Restart recovery.** After a terminal restart or VPS reboot, the EA reconstructs its full state from the account history — open positions, exit timers, breaker status, sizing state — and resumes managing exactly where it left off.
- **Clean separation.** The Blitz identifies its own trades by unique markers and never modifies or closes positions belonging to you or other EAs.

## Account-level protections you control

All off by default, all in plain settings (see [settings.md](settings.md)):

- **Daily loss limit** and **daily drawdown limit**, in money or percent. On breach, the EA closes only its own positions and stands down until the next day — a prop-firm-style brake.
- **News filter.** Blocks new entries around high-impact economic events, using MT5's built-in calendar.
- **Scheduled exits.** Optional daily flat time and Friday close, at broker server time.
- **Time blocks.** Up to four daily windows with no new entries.
- **Risk-based sizing.** Fixed lots by default; optionally risk a fixed percentage of balance per trade, with hard caps on both percentage and lot size.

## The monitoring philosophy

Most EAs show you a profit number. The Blitz shows you whether the strategy is *behaving like itself*. Two panels (documented field by field in [dashboard.md](dashboard.md)) continuously measure:

- the **current market regime** on the daily chart (trend structure, volatility percentile),
- your **live results versus the strategy's historical reference profile** — expectancy, profit-factor drift, stagnation length, losing-streak percentile, equity-curve smoothness, value-at-risk,
- and the **divergence between your live execution and the simulated benchmark**.

The point: drawdowns and losing streaks are normal, and panicking at them is how good systems get abandoned. The panels tell you whether what you're seeing sits inside the strategy's historical envelope (keep calm) or outside it (worth your attention) — an early-warning system rather than a scoreboard.

## What The Blitz does NOT do

- No martingale, grid, or loss-averaging — ever.
- No re-optimization or behavior changes at runtime.
- No interference with your manual trades or other EAs.
- No trading through your configured limits, schedules, or news windows.
