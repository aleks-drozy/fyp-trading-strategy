# FYP BOT 1 — Algorithmic Trading Strategy

**Aleksandrs Drozdovs | 22514179 | Maynooth University FYP 2026**

An automated trading strategy built in PineScript v6 on TradingView, targeting the NASDAQ-100 E-mini futures contract (NQ1!) during the New York morning session. The strategy uses two Smart Money Concepts: Inverse Fair Value Gaps (IFVG) and Change in State of Delivery (CISD) as a double confirmation entry system.

---

## Backtest Results (NQ1!, Jan 2025 – Feb 2026)

| Metric | Value |
|---|---|
| Total Trades | 72 |
| Win Rate | 56.94% |
| Net P&L | +$28,400 |
| Profit Factor | 1.703 |
| Max Drawdown | 0.95% |
| Expectancy | +$211 per trade |

---

## Repository Structure

```
/
├── FYP_BOT_1.pine          # Main strategy — PineScript v6
├── equity_curve.html       # Interactive equity curve visualisation tool
├── backtests/
│   ├── NQ1_optimised.csv   # Trade log — NQ1! Jan 2025–Feb 2026 (optimised)
│   ├── NQ1_OOS_2023.csv    # Trade log — NQ1! Jan 2023–Dec 2024 (out-of-sample)
│   └── NQ1_optimised.png       # TradingView equity curve screenshot
└── README.md
```

---

## How the Strategy Works

### 1. IFVG Detection
A Fair Value Gap (FVG) is a 3-candle price imbalance where buyers or sellers moved price so aggressively that the opposite side had no time to participate. When price closes across the fair value gap, it becomes an **Inverse Fair Value Gap (IFVG)** the gap's bias flips, signalling that the prior imbalance has been absorbed and price is likely to continue in the new direction.

- **Bullish IFVG**: price closes above a bearish FVG from below → bullish signal
- **Bearish IFVG**: price closes below a bullish FVG from above → bearish signal

### 2. CISD Detection
A Change in State of Delivery (CISD) identifies a structural shift in market direction. The algorithm detects pullbacks by monitoring candle directionality, tracks the extreme price reached during the pullback, and marks a CISD level when price subsequently breaks through that extreme.

- **Bullish CISD**: price closes above a prior bearish swing high → market shifted bullish
- **Bearish CISD**: price closes below a prior bullish swing low → market shifted bearish

### 3. Double Confirmation Entry
A trade is only entered when **both** IFVG and CISD signals align in the same direction at the same time. This eliminates the majority of false signals that would arise from using either indicator alone.

### 4. Liquidity Sweep Filter
Before entering, the strategy requires a recent **liquidity sweep** on the 1-minute chart: a wick beyond the recent swing high/low that closes back inside. This confirms that pending stop orders have been triggered and absorbed by institutional participants, making a genuine directional move more likely.

### 5. Risk Management
- **Stop loss**: placed at the swing low (longs) or swing high (shorts) using an 8-bar lookback
- **Take profit**: stop distance × 1.5 R:R ratio
- **Session**: entries restricted to 09:32–10:00 New York time only
- **Max trades**: 1 per session

---

## How to Use — TradingView

1. Open [TradingView](https://www.tradingview.com) and navigate to the Pine Script Editor
2. Copy the contents of `FYP_BOT_1.pine` into the editor
3. Click **Add to chart**
4. Open the **Strategy Tester** tab to view backtest results
5. Use the settings panel to adjust parameters (R:R ratio, session window, swing lookback)

To export trade logs for the equity curve tool:
1. In Strategy Tester → **List of Trades** tab
2. Click the export icon (top right) → **Export trades to CSV**

---

## How to Use: Equity Graph Tool

1. Open `equity_curve.html` in any modern browser (Chrome, Firefox, Safari)
2. Click **Upload CSV** and select a TradingView strategy export
3. The tool will render:
   - Interactive equity graph with per-trade colouring
   - Summary statistics (win rate, profit factor, drawdown, expectancy)
   - Monthly P&L breakdown
   - Full scrollable trade log

Runs on browser.

---

## Optimised Parameters

| Parameter | Value |
|---|---|
| Session Window | 09:32–10:00 NY |
| Risk/Reward Ratio | 1.5 |
| Swing Lookback | 8 bars |
| Sweep Lookback | 5 bars |
| Sweep Expiry | 5 bars |
| Max Trades/Day | 1 |

---

## Parameter Optimisation Summary

The following filters were tested. Only the 1-minute liquidity sweep improved performance:

| Filter | Result |
|---|---|
| HTF EMA Trend | Neutral |
| Minimum FVG Size | Harmful — reduced trades and P&L |
| Strong Candle | Harmful — removed valid signals |
| Volume Filter | Harmful — TradingView futures volume unreliable |
| 1-min Liquidity Sweep | ✅ Beneficial — +7.76% win rate, +0.311 profit factor |

---

## Out-of-Sample Results (NQ1!, Jan 2023 – Dec 2024)

The strategy underperforms in low-volatility, slow-trending markets. The 2023–2024 period produced a net loss, which is consistent with the strategy's design: it requires sharp, impulsive price moves to generate clean IFVG and CISD patterns. When those conditions are absent, the edge disappears.

| Metric | In-Sample 2025–26 | Out-of-Sample 2023–24 |
|---|---|---|
| Win Rate | 56.94% | 36.27% |
| Profit Factor | 1.703 | 0.855 |
| Net P&L | +$28,400 | -$15,650 |

---

## Supervisor

Dr. Phil Maguire — Department of Computer Science, Maynooth University
