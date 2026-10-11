# The Blitz — User Manual

**Version 1.06** · MetaTrader 5 Expert Advisor · Gold (XAUUSD) · Default chart timeframe: H1

The Blitz is an automated trading system for MetaTrader 5 that trades Gold (XAUUSD) with two independent strategy engines. Each engine uses a machine-learning model (gradient-boosted decision trees, trained on 2018–2024 data) to score every completed bar, and only trades when the predicted edge is strong enough. Position sizing, exits, trailing stops, and protective limits are all handled automatically.

This manual explains how to install the EA, what every setting does, how to read the two on-chart panels, and answers the questions most users ask.

> **Risk warning.** Trading leveraged products such as Gold/CFDs carries a high risk of losing money. Past performance — including backtests and the historical references shown on the dashboard — does not guarantee future results. Never trade money you cannot afford to lose, and always test on a demo account first.

---

## Table of contents

1. [Quick start](#1-quick-start)
2. [Requirements](#2-requirements)
3. [How The Blitz trades](#3-how-the-blitz-trades)
4. [Default settings at a glance](#4-default-settings-at-a-glance)
5. [The two panels on your chart](#5-the-two-panels-on-your-chart)
6. [Detailed documentation](#6-detailed-documentation)
7. [FAQ and troubleshooting](#7-faq-and-troubleshooting)
8. [Files The Blitz creates](#8-files-the-blitz-creates)

---

## 1. Quick start

1. Open **MetaTrader 5** and log in to your account (a **hedging** account is required — see below).
2. Copy the compiled file `The_Blitz_v1_06.ex5` into `MQL5/Experts/` (menu: *File → Open Data Folder → MQL5 → Experts*). If you received the source code instead, see *Compiling from source* below.
3. Open a **Gold (XAUUSD)** chart and set the chart timeframe to **H1**.
4. Drag **The Blitz** from the Navigator onto the chart.
5. In the settings dialog, leave everything at its default for the standard configuration (0.01 lots per engine, all optional protections off).
6. Make sure the **Algo Trading** button in the toolbar is green, and the "Allow algorithmic trading" box is ticked in the EA's Common tab.
7. Press OK. The dashboard appears in the top-left corner and the Quant Panel to its right.

That's it. Both engines are enabled by default and will trade automatically when their models see a qualifying setup.

### Compiling from source (only if you have the .mq5 file)

The source file `The_Blitz_v1_06_Compact.mq5` needs the image file `blitz_header.bmp` (from the v1.05 developer bundle) in the same folder to compile — it is embedded into the EA at compile time. Place both files together, open the .mq5 in MetaEditor, and press **Compile**. The resulting `.ex5` is fully self-contained; the image is not needed afterwards.

---

## 2. Requirements

| Requirement | Detail |
|---|---|
| Terminal | MetaTrader 5 (desktop) |
| Account type | **Hedging** (the EA refuses to start on netting accounts) |
| Symbol | Gold — typically `XAUUSD`; attach to your broker's Gold symbol |
| Chart timeframe | **H1 recommended** (default). The engines read their own H1 and H4 data internally, so other chart timeframes work, but H1 is the tested configuration. |
| History | At least ~600 H1/H4 bars and 220 daily bars available on the symbol for signals and Market Health |
| Spread | Entries are skipped when the spread exceeds 50 points (default) |
| VPS | Not required, but recommended for uninterrupted 24/5 operation |
| Internet (news filter) | The optional news filter uses the built-in MT5 economic calendar and works **live only**, not in the Strategy Tester |

---

## 3. How The Blitz trades

### Two engines ("Marks"), one chart

The Blitz runs two independent strategy engines at the same time. Each has its own model, timeframe, exits, and magic number, so its trades are fully separated from the other's — and from any trades you or other EAs place.

| | **Mark 1** | **Mark 2** |
|---|---|---|
| Timeframe | H1 | H4 |
| Model | Challenger rank-1 H1 model | Challenger rank-1 H4 model |
| Entry threshold (predicted edge) | ≥ 0.10 R | ≥ 0.05 R |
| Take profit | 4 × ATR | 3 × ATR |
| Stop loss | 3 × ATR | 2 × ATR |
| Time stop | 24 bars (≈ 24 hours) | 6 bars (≈ 24 hours) |
| Trailing stop | Yes — activates after +1.0× initial risk, trails 1.0× behind | Yes — activates after +0.75× initial risk, trails 0.5× behind |
| Circuit breaker | Off | 3 consecutive losses → pause new entries for 5 days |
| Session filter | Off | No entries at server hours 04, 10, 21, 22 |
| Max simultaneous positions | 2 | 2 |
| Stop-distance cap | Skip trade if stop wider than $50/oz | Skip trade if stop wider than $80/oz |
| Default size | 0.01 lots | 0.01 lots |

These engine internals are **frozen** — they are part of the verified research model and are not user settings. What you control is position sizing and the optional protections (see [docs/settings.md](docs/settings.md)).

### What the model actually does

On every completed bar, the engine computes 17 features from the last 600 bars: momentum (RSI-based), price moves over 1/3/6/12/48 bars, bar shape, volatility regime, trend efficiency, time-of-day and day-of-week cycles, and the current spread cost. Two small gradient-boosted tree ensembles (100 trees each, one for long, one for short) then predict the expected net outcome of a trade **in multiples of risk (R)**. If the better of the two predictions clears the engine's threshold, the EA opens a position in that direction with a fixed ATR-based stop and target.

Plain version: *the EA only trades when its model expects the trade to be worth more than a fixed fraction of the money risked, and every trade has a predefined stop loss, take profit, and maximum lifetime.*

### Every trade is protected

- **Stop loss and take profit** are placed with the order, always.
- **Time stop**: a trade that has not resolved within ~24 hours is closed at market.
- **Trailing stop**: once a trade moves far enough in profit, the stop follows price and locks in gains.
- **Spread gate**: no entries when the spread is abnormally wide.
- **Stop-distance cap**: no entries when volatility makes the required stop unusually expensive.
- **Circuit breaker (Mark 2)**: after 3 losses in a row, that engine rests for 5 days.
- **Optional account-level protections** you can switch on: daily loss/drawdown limits, news filter, end-of-day and Friday exits, and custom no-entry time blocks.

### What "R" means

Throughout the EA and its panels, **1 R = the amount of money risked on a trade** (the distance from entry to stop loss, in money). A trade closed at +2 R earned twice what it risked; −1 R is a full stop-out. Thinking in R lets the dashboard compare performance regardless of lot size.

---

## 4. Default settings at a glance

The EA ships ready for **Gold on H1** with conservative fixed sizing:

- Both engines **enabled**, 0.01 lots each
- Fixed lot sizing (auto risk-based sizing **off**)
- Dynamic scaling **off**
- Daily limits **off**
- News filter **off**
- Scheduled exits **off** (no forced end-of-day or Friday close)
- Time blocks **off**
- Trailing stops **on** for both engines
- Dashboard and Quant Panel **shown**, panels at 80% size
- Performance Observer (historical mock) **on**, 120-day initial lookback

Every input is documented, field by field, in **[docs/settings.md](docs/settings.md)**.

---

## 5. The two panels on your chart

### The Blitz dashboard (left panel)

Shows what the EA is doing right now:

- **MARK 1 / MARK 2 cards** — engine state (ACTIVE / PAUSED / OFF), open positions, current lot size or risk mode
- **MARKET HEALTH** — a structural read of the Gold daily chart (bull fan, distance above the 200-day average, full-bull days) with a CLEAR / NEUTRAL / CAUTION verdict. *Informational only — it never blocks trades.*
- **PERFORMANCE BRIDGE** — Profit Factor, Average R, and worst Loss Run over your most recent 30 closed results, compared against frozen research references, with a verdict line underneath
- **Footer** — daily limit status, news filter status, broker clock

### The Quant Panel (right panel)

A deeper statistical monitor with two columns: **MOCK** (a historical simulation of the exact current configuration on your broker's recent data) and **LIVE** (your actual executed results on this account). Ten rows covering expectancy, profit-factor drift, stagnation, losing-streak survival, Kelly sizing, equity-curve quality, daily Value-at-Risk, engine diversification, volatility regime, and live-vs-mock divergence.

Every field, color, and verdict on both panels is explained in plain language in **[docs/dashboard.md](docs/dashboard.md)**.

---

## 6. Detailed documentation

| Document | Contents |
|---|---|
| [docs/how-it-works.md](docs/how-it-works.md) | The strategy in depth: models, entries, exits, trailing, breakers, sizing modes, and all protective systems |
| [docs/settings.md](docs/settings.md) | Every input parameter: what it does, default, and recommended use |
| [docs/dashboard.md](docs/dashboard.md) | Every statistic on the dashboard and Quant Panel, and how to interpret it |
| [docs/faq.md](docs/faq.md) | FAQ and troubleshooting: "why is it not trading", backtesting, prop firms, VPS, updates |

---

## 7. FAQ and troubleshooting

Short answers to the most common questions:

- **Nothing is trading?** The model trades selectively — quiet days are normal. Check the Mark cards: PAUSED means the circuit breaker is resting; OFF means the engine is disabled. Also check spread, the news line, and any time blocks you enabled.
- **Which account size?** With default 0.01 lots on Gold, even small accounts can run it, but your broker's margin for Gold is the binding constraint. Use auto risk sizing if you want risk to scale with balance.
- **Can I run it with other EAs?** Yes. The Blitz only touches positions carrying its own magic numbers (260925 / 260926).
- **Backtesting?** Works in the Strategy Tester; the news filter and the historical mock are automatically disabled there.

The full list is in **[docs/faq.md](docs/faq.md)**.

---

## 8. Files The Blitz creates

Stored in the terminal's `MQL5/Files` folder, per account + server + symbol:

| File | Purpose |
|---|---|
| `Blitz_<account>_<server>_<symbol>_live_results.csv` | Ledger of your closed Blitz trades (drives the LIVE statistics) |
| `Blitz_<account>_<server>_<symbol>_dashboard_state_v104.csv` | Saved panel state, restored after terminal restarts |
| `Blitz_mock_v105_<account>_<server>_<symbol>.csv` | Cached historical mock stream (drives the MOCK statistics) |
| `Blitz_deals.csv` | Diagnostic deal log with all costs (when `LogDeals` is on) |
| `Blitz_H1_parity.csv`, `Blitz_H4_parity.csv` | Feature-parity diagnostics (only when `LogFeatureParity` is on) |

Deleting these files is safe — the EA rebuilds them — but your LIVE statistics history restarts from scratch.

---

*The Blitz v1.06 · This manual describes the retail build exactly as shipped: frozen strategy engines, configurable risk and protections, and full on-chart transparency.*
