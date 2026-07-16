# Features

This feature matrix describes behavior present in the current source.

## Foundation Features

| Feature | Status | Notes |
| --- | --- | --- |
| Adaptive MTF trend stack | Live gate | Auto-selects three MTFs or uses manual MTFs. Requirements vary by preset. |
| External market structure | Live gate and visual | Uses external pivot highs/lows to classify HH/HL or LH/LL and CHoCH. |
| Smart money zones | Live through pattern and visuals | OB/FVG logic feeds patterns and visuals. The master switch `i_useSMC` is not wired. |
| BTC correlation gate | Live gate | Auto-bypasses when ticker contains `BTC`. |
| HTF S/R confluence | Live gate | Uses latest two HTF support and resistance pivots. |

## Confluence Features

| Feature | Status | Notes |
| --- | --- | --- |
| RSI confirmation | Live confluence | Includes RSI zones, overbought/oversold, and simple divergence. |
| MACD momentum shift | Live confluence | Uses crossover/crossunder and histogram acceleration/state. |
| Volume spike validation | Live confluence | Requires current volume above SMA volume times threshold. |
| Pattern qualifier | Live core gate | Requires breakout retest, sweep reversal, OB tap, or FVG fill. |
| Session timing | Live confluence and gate | London/NY sessions, optional weekend skip. |
| Volatility regime | Live confluence | ATR% and ADX threshold. |
| CVD proxy divergence | Live confluence | Uses candle-direction volume delta proxy. |
| Funding rate gate | Live optional gate | Manual input, disabled by default. |
| Open Interest divergence | Live optional gate | Uses OI symbol volume or close-volume proxy, disabled by default. |
| News/event blackout | Live gate | Auto recurring windows plus manual windows. |
| Crypto sentiment engine | Live gate and confidence boost | Combines BTC.D, USDT.D, TOTAL, TOTAL3. |

## Signal And Scoring Features

| Feature | Status | Notes |
| --- | --- | --- |
| Preset-based signal strictness | Live | Strict, Balanced, Aggressive, Scalping, Aggressive Scalping, Custom. |
| Confluence count | Live | Counts enabled RSI, MACD, volume, structure, session, CVD, HTF S/R, volatility. |
| Weighted confidence score | Live | MTF, pattern, candle, confluence count, HTF S/R, sentiment. |
| Long/short discipline switches | Live | User can disable longs or shorts. |
| Backtest date range | Live | Optional date filter. |
| Blocker diagnostic | Live dashboard | Shows first missing condition for long and short paths. |

## Risk And Execution Features

| Feature | Status | Notes |
| --- | --- | --- |
| Adaptive ATR sizing | Live | Stop multiplier changes with volatility and scalping preset. |
| Structure-aware stop placement | Live | Uses OB or external pivot when available, with ATR fallback bound. |
| Equity-based quantity | Live | Risk amount divided by stop distance. |
| Minimum reward:risk gate | Live | Threshold comes from preset or Custom input. |
| Partial TP plus breakeven | Live optional execution | Submits BE stop after +1R when enabled. |
| Multi-target TP | Live optional execution | TP1, TP2, and runner trail. |
| Daily loss circuit breaker | Live gate | Blocks new signals after configured daily drawdown. |
| Daily trade cap | Live gate | Preset or Custom max trades per day. |
| Cooldown bars | Live gate | Preset or Custom cooldown after signals. |
| Win/loss streak sizing | Live | Reduces or increases size after configured streaks. |
| Hybrid accumulation add | Live optional execution | Attempts one add at +1R when enabled. |

## Analytics Features

| Feature | Status | Notes |
| --- | --- | --- |
| Sharpe approximation | Calculated and displayed | Dashboard backtest row includes Sharpe approximation. |
| Z-score | Calculated only | Not displayed or used in gates. |
| Bollinger %B | Calculated only | Not displayed or used in gates. |
| Hurst proxy | Calculated only | Not displayed or used in gates. |
| Expected value | Calculated only | Not displayed or used in gates. |
| Kelly fraction | Calculated only | Not displayed or used in gates. |
| Sortino approximation | Calculated only | Not displayed or used in gates. |
| Calmar ratio | Calculated only | Not displayed or used in gates. |
| Recovery factor | Calculated only | Not displayed or used in gates. |
| Expectancy per trade | Calculated only | Not displayed or used in gates. |
| Equity curve filter | Calculated only | `equityCurveOK` is not used by final signals. |
| Max DD shutdown | Calculated only | `maxDDshutdown` is not used by final signals. |
| Anti-revenge cooldown | Calculated only | `antiRevengeOK` is not used by final signals. |

## Output Features

| Feature | Status | Notes |
| --- | --- | --- |
| Full dashboard | Live visual | 27 rows, persistent table. |
| Giant signal banner | Live visual | Shows last signal for 50 bars or while trade is open. |
| Visual signals | Live visual | Long/short markers, labels, bar colors, flashes. |
| Zone boxes | Live visual | OB and FVG boxes on last bar. |
| EMA plots | Live visual | EMA20, EMA50, EMA200. |
| Structure visuals | Live visual | Pivot markers, dashed external levels, CHoCH labels. |
| HTF S/R lines | Live visual | Latest two support and resistance levels. |
| SL/TP/trail plots | Live visual | Active position levels. |
| Plain text alerts | Live alert | Uses `alert()` and `alertcondition()`. |
| JSON webhook alerts | Live alert | User-selectable alert format. |

## Input Groups

The script exposes these input groups:

- Preset
- Master Switches
- Custom Overrides
- Adaptive MTF Stack
- Structure / SMC
- Liquidity Filter
- HTF S/R Confluence
- RSI
- MACD
- Volume + CVD
- BTC Correlation
- Funding & OI
- Crypto Sentiment
- Session Filter
- Volatility Regime
- News / Event Blackout
- Risk Management
- Multi-Target TP
- Daily Drawdown
- Streak Sizing
- Discipline
- Backtest Range
- Visualization
- Alerts
- Accumulation Strategy

## Current Limitations

- `i_useSMC` does not disable OB/FVG pattern participation.
- `i_htfsrLookback` does not affect HTF S/R behavior.
- Some calculated analytics are not visible in the dashboard.
- Some calculated risk controls are not wired into signal gates.
- The script is monolithic, so module boundaries are conceptual rather than function-level separations.
