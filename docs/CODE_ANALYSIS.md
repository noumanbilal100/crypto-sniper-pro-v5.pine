# Code Analysis Report

Source analyzed: `crypto-sniper-pro-v5.pine`  
Pine version: v6  
Script type: `strategy`, overlay enabled  
Source size: 1,645 lines  
Baseline SHA-256 before this report: `70603E70DF701ADC01CE8BBA05D78DD99D4A63410241A4B69103954E81D1B79C`

This report reverse engineers the current implementation. It does not propose or apply trading logic changes.

## 1. Overall Architecture

The script is a monolithic Pine Strategy with a staged pipeline. There are no user-defined functions; module boundaries are implemented through comment sections and variable naming.

High-level architecture:

```text
strategy declaration
  -> preset and input resolution
  -> adaptive timeframe selection
  -> core market state
  -> trend, structure, SMC, momentum, volume, sentiment, and risk calculations
  -> confluence counts and confidence scores
  -> final longSignal / shortSignal gates
  -> strategy entries and exits
  -> analytics calculations
  -> visualization, dashboard, and alerts
```

The strategy is designed around confirmed-bar execution:

- `calc_on_every_tick=false`
- `process_orders_on_close=true`
- most signal-critical conditions include `barstate.isconfirmed`
- alerts fire with `alert.freq_once_per_bar_close`

The major architectural layers are:

| Layer | Line range | Role |
| --- | ---: | --- |
| Strategy declaration | 42-59 | TradingView strategy settings and limits. |
| Presets and inputs | 61-291 | User configuration and derived preset thresholds. |
| Adaptive timeframe selection | 293-305 | Automatic or manual MTF resolution. |
| Core engines | 307-718 | ATR, EMA, MTF, structure, liquidity, OB, FVG, HTF S/R, RSI, MACD, volume, CVD, BTC, sentiment, funding, OI, sessions, volatility, news. |
| Pattern and Wyckoff logic | 719-798 | Candle confirmation, setup patterns, phase classification. |
| Signal gate | 800-950 | Confluence counts, confidence, risk checks, cooldown, daily caps, final signals. |
| Execution | 952-1019 | Entries, exits, BE stop, trailing stop. |
| Analytics | 1021-1155 | Backtest/statistical metrics and optional accumulation pyramid. |
| Visuals | 1157-1452 | Plots, labels, boxes, lines, signal banner. |
| Dashboard | 1454-1619 | Persistent status table. |
| Alerts | 1621-1645 | Plain text, JSON, alertcondition outputs. |

## 2. Execution Flow From Top To Bottom

1. The script declares a strategy with fixed backtest assumptions and chart object limits.
2. The preset selector maps `i_preset` into active thresholds: minimum confirmation count, confidence percent, cooldown, minimum R:R, and maximum daily trades.
3. Input groups declare all runtime configuration.
4. Text and label size inputs are converted into Pine size enums.
5. The adaptive timeframe engine maps the current chart timeframe into three higher timeframes unless manual mode is selected.
6. ATR and broad volatility state are calculated.
7. The LTF EMA engine builds EMA20/EMA50/EMA200 trend state and slope.
8. The MTF engine requests EMA20/EMA50 from three higher timeframes and scores bullish/bearish alignment.
9. External pivots update market structure state and detect BOS/CHOCH.
10. Liquidity sweeps are detected against confirmed external pivots.
11. OB and FVG engines track the latest active bullish and bearish zones.
12. HTF S/R stores recent HTF pivots and checks proximity.
13. RSI, MACD, volume, and CVD confirmation engines run.
14. BTC, sentiment, funding, and OI gates run.
15. Session, weekend, volatility regime, and news blackout gates run.
16. Candle and pattern engines determine whether an actionable setup exists.
17. Wyckoff phase state is calculated for context and visuals.
18. Confluence counts are computed for long and short.
19. Preset-specific trend, MTF, and candle requirements are resolved.
20. `longCore` and `shortCore` are built from trend, setup, filter, and confirmed-bar requirements.
21. Confidence scores are calculated and capped at 100.
22. Stop, target, reward:risk, and quantity calculations are prepared.
23. Daily drawdown, win/loss streak, cooldown, daily cap, and date filters update.
24. `longSignal` and `shortSignal` are finalized.
25. Flat-position entries are submitted when final signals fire.
26. Exits are submitted for multi-target or single-target mode.
27. Optional breakeven exits and trailing closes manage open positions.
28. Analytics are computed.
29. Optional accumulation pyramid entries are submitted at +1R.
30. Visual elements, dashboard, signal banner, and alerts are rendered or emitted.

## 3. Every Input Group

| Group | Lines | Inputs |
| --- | ---: | --- |
| Preset | 64-67 | `i_preset` |
| Master Switches | 85-104 | MTF, structure, SMC, BTC, HTF S/R, RSI, MACD, volume, session, volatility, CVD, funding, OI, news, sentiment, partial TP, multi-TP, daily DD, streak sizing |
| Custom Overrides | 107-112 | custom confirmations, confidence, cooldown, R:R, daily max |
| Adaptive MTF Stack | 122-127 | MTF mode and three manual TFs |
| Structure / SMC | 130-135 | pivot length, BOS ATR distance, OB impulse threshold, OB max age, FVG minimum ATR |
| Liquidity Filter | 138-140 | sweep proximity ATR and prior-trend requirement |
| HTF S/R Confluence | 143-147 | HTF S/R timeframe, lookback, pivot length, proximity ATR |
| RSI | 150-154 | length, OB, OS, divergence lookback |
| MACD | 157-160 | fast, slow, signal |
| Volume + CVD | 163-167 | volume MA length, spike threshold, CVD smoothing, CVD divergence lookback |
| BTC Correlation | 170-172 | BTC reference symbol and trend timeframe |
| Funding & OI | 175-179 | manual funding, OI symbol, OI proxy toggle |
| Crypto Sentiment | 182-190 | BTC.D, USDT.D, TOTAL, TOTAL3, sentiment timeframe, extreme threshold |
| Session Filter | 193-197 | London, NY, timezone, weekend skip |
| Volatility Regime | 200-203 | ATR percent minimum, ADX length, ADX minimum |
| News / Event Blackout | 206-221 | recurring event toggles/windows and three manual event windows |
| Risk Management | 224-229 | risk percent, stop buffer, adaptive ATR toggle, low/high volatility ATR multipliers |
| Multi-Target TP | 232-237 | TP1 R, TP1 percent, TP2 R, TP2 percent, runner trail ATR |
| Daily Drawdown | 240-241 | max daily DD percent |
| Streak Sizing | 244-248 | loss-streak and win-streak thresholds and sizing adjustments |
| Discipline | 251-253 | allow long, allow short |
| Backtest Range | 256-259 | date filter toggle, start, end |
| Visualization | 262-282 | EMA, structure, OB, FVG, HTF S/R, Wyckoff, phase labels, signals, SL/TP, dashboard, dashboard position, text sizes |
| Alerts | 290-291 | alert format |
| Accumulation Strategy | 1130-1133 | optional +1R pyramid add and add size |

Note: The accumulation inputs are declared later than the main input block, inside the advanced analytics/execution section.

## 4. Every Indicator And Filter Used

| Component | Lines | Type | Used in signals |
| --- | ---: | --- | --- |
| ATR 14 and ATR 50 | 310-314 | volatility state | yes, indirectly through stops, thresholds, volatility state |
| EMA20/EMA50/EMA200 | 319-331 | trend | yes |
| MTF EMA20/EMA50 | 333-346 | multi-timeframe trend | yes |
| External pivots | 348-380 | market structure | yes |
| Liquidity sweeps | 382-394 | liquidity/SMC | yes, through patterns and Wyckoff |
| Order blocks | 396-430 | SMC zone | yes, through patterns and stops |
| Fair value gaps | 432-458 | SMC imbalance | yes, through patterns |
| HTF S/R | 460-480 | HTF confluence | yes |
| RSI and RSI divergence | 482-499 | momentum/mean reversion | yes |
| MACD | 501-509 | momentum | yes |
| Volume spike | 511-514 | confirmation | yes |
| CVD proxy | 516-531 | volume pressure/divergence | yes |
| BTC trend | 533-541 | market filter | yes |
| Crypto sentiment composite | 543-628 | market regime/filter | yes |
| Funding gate | 630-637 | crowding filter | optional |
| OI proxy/divergence | 639-650 | participation filter | optional |
| Session filter | 652-656 | time filter | yes, if enabled |
| ATR% + ADX | 658-672 | volatility regime | yes |
| News blackout | 674-718 | event risk filter | yes |
| Candle confirmation | 719-725 | trigger confirmation | yes |
| Pattern engine | 727-737 | setup qualifier | yes |
| Wyckoff phase | 739-798 | regime/visual context | no final gate |
| Sharpe and analytics | 1021-1126 | performance/statistics | mostly no |

## 5. Trend Engine

The trend engine is split into LTF trend, MTF trend, and contextual filters.

### LTF EMA Trend

Lines 318-331 calculate:

- `ema20`
- `ema50`
- `ema200`
- `emaBull`: EMA20 > EMA50 > EMA200
- `emaBear`: EMA20 < EMA50 < EMA200
- `emaSlope`: five-bar change in EMA50
- `ltfTrendBull`: bullish EMA stack and rising EMA50
- `ltfTrendBear`: bearish EMA stack and falling EMA50
- `trendLabel`: display string used by dashboard and tooltips

Preset impact:

- Strict/Balanced style requires `ltfTrendBull` or `ltfTrendBear`.
- Aggressive/scalping style accepts `emaBull` or `emaBear` without slope.

### MTF Trend

Lines 333-346 request EMA20/EMA50 on three selected higher timeframes. Each timeframe receives a bias:

- `1` bullish
- `-1` bearish
- `0` neutral

The strategy counts bullish and bearish MTF scores. Preset logic at lines 837-839 defines how many MTFs must agree.

### Trend Consumers

Trend state feeds:

- `trendPassLong`
- `trendPassShort`
- `longCore`
- `shortCore`
- Wyckoff markup/markdown classification
- confidence score
- dashboard
- signal tooltips

## 6. Smart Money Concepts Engine

The SMC layer combines external structure, liquidity sweeps, OBs, FVGs, HTF S/R, setup patterns, and Wyckoff phase context.

Live SMC consumers:

- `patternBull`
- `patternBear`
- `longSL`
- `shortSL`
- confluence counts
- confidence
- dashboard zone rows
- chart boxes/labels

Important implementation note: `i_useSMC` is declared at line 88 but is not referenced in active signal formulas. OB/FVG behavior remains live through the pattern engine even when that master switch is disabled.

## 7. Liquidity Detection

Lines 382-394 define liquidity sweeps against confirmed external pivots.

High sweep:

- a valid `lastExtHigh` exists
- current high exceeds it
- current close returns below it
- high is within `i_liqNearATR * ATR` of the pivot
- optional prior trend movement condition passes
- bar is confirmed

Low sweep:

- a valid `lastExtLow` exists
- current low trades below it
- current close returns above it
- low is within `i_liqNearATR * ATR` of the pivot
- optional prior trend movement condition passes
- bar is confirmed

The prior trend movement test is:

```text
abs(close - close[10]) > ATR * 3
```

Liquidity sweep outputs:

- `sweepHigh`
- `sweepLow`

Consumers:

- sweep reversal patterns
- Wyckoff manipulation phase
- visual sweep markers

## 8. Order Blocks

Lines 396-430 implement a latest-zone OB model.

Bullish OB creation:

- current candle is a bullish impulse: `close - open > ATR * i_obImpulse`
- current candle closes bullish
- previous candle was bearish
- previous candle body becomes the OB

Bearish OB creation:

- current candle is a bearish impulse: `open - close > ATR * i_obImpulse`
- current candle closes bearish
- previous candle was bullish
- previous candle body becomes the OB

State:

- `bullOB_top`, `bullOB_bot`, `bullOB_bar`, `bullOB_fresh`
- `bearOB_top`, `bearOB_bot`, `bearOB_bar`, `bearOB_fresh`

Invalidation:

- bullish OB invalidates when price trades to or below the OB bottom
- bearish OB invalidates when price trades to or above the OB top

Age filter:

- OB remains valid only while age is less than or equal to `i_obMaxBars`

Consumers:

- `inBullOB`
- `inBearOB`
- pattern engine
- stop placement
- dashboard zone row
- OB boxes and labels

## 9. Fair Value Gaps

Lines 432-458 implement a latest-zone FVG model.

Bullish FVG:

- current low is above high from two bars ago
- gap size is at least `ATR * i_fvgMinATR`

Bearish FVG:

- current high is below low from two bars ago
- gap size is at least `ATR * i_fvgMinATR`

State:

- `bullFVG_top`
- `bullFVG_bot`
- `bullFVG_active`
- `bearFVG_top`
- `bearFVG_bot`
- `bearFVG_active`

Invalidation:

- bullish FVG deactivates when low reaches the bullish FVG bottom
- bearish FVG deactivates when high reaches the bearish FVG top

Consumers:

- `inBullFVG`
- `inBearFVG`
- pattern engine
- dashboard zone row
- FVG boxes and labels

## 10. Market Structure: BOS And CHOCH

External pivots are detected with:

- `ta.pivothigh(high, i_pivotLen, i_pivotLen)`
- `ta.pivotlow(low, i_pivotLen, i_pivotLen)`

The script stores latest and previous confirmed external highs/lows.

Bullish structure:

- latest external high > previous external high
- latest external low > previous external low

Bearish structure:

- latest external high < previous external high
- latest external low < previous external low

BOS logic:

- `bosBull`: confirmed close above last external high plus ATR distance
- `bosBear`: confirmed close below last external low minus ATR distance

CHOCH logic:

- `chochBull`: bearish structure followed by bullish BOS
- `chochBear`: bullish structure followed by bearish BOS

Structure consumers:

- confluence counts
- breakout/retest patterns
- liquidity sweep anchors
- stop placement
- dashboard
- chart labels and pivot markers

## 11. Volume Logic

Volume appears in three distinct systems.

### Volume Spike

Lines 511-514:

- `volMA = ta.sma(volume, i_volLen)`
- `volSpike = volume > volMA * i_volSpike`
- `volConfirm = not i_useVol or (volSpike and barstate.isconfirmed)`

Used by:

- confluence counts
- Wyckoff manipulation confirmation
- dashboard volume row

### CVD Proxy

Lines 516-531:

- bullish candle volume contributes positive volume
- bearish candle volume contributes negative volume
- EMA of bullish minus bearish volume creates `cvdDelta`

CVD divergence:

- bullish CVD divergence requires price at a close low while CVD is above its previous low
- bearish CVD divergence requires price at a close high while CVD is below its previous high

### OI Proxy

Lines 639-650:

- pulls configured OI symbol close and volume
- proxy is either `close * volume` or volume
- price rising requires OI proxy rising for longs
- price falling requires OI proxy rising for shorts

## 12. Momentum Filters

### RSI

Lines 482-499:

- bullish zone: RSI between 45 and 70
- bearish zone: RSI between 30 and 55
- oversold: RSI below input OS
- overbought: RSI above input OB
- bullish divergence: price lower low with RSI higher low and RSI below 40
- bearish divergence: price higher high with RSI lower high and RSI above 60

### MACD

Lines 501-509:

- bullish shift: MACD crossover or improving histogram while line is above signal after negative histogram
- bearish shift: MACD crossunder or weakening histogram while line is below signal after positive histogram
- bullish state: MACD line above signal and histogram positive
- bearish state: MACD line below signal and histogram negative

Momentum consumers:

- confluence counts
- dashboard
- signal eligibility through minimum confluence threshold

## 13. Multi-Timeframe Engine

Lines 293-305 map chart timeframe into MTFs:

| Chart timeframe bucket | MTF1 | MTF2 | MTF3 |
| --- | --- | --- | --- |
| <= 1m | 5m | 15m | 60m |
| <= 5m | 15m | 60m | 240m |
| <= 15m | 60m | 240m | D |
| <= 60m | 240m | D | W |
| <= 240m | D | W | M |
| <= 1D | W | M | 3M |
| higher | W | M | 12M |

Manual mode replaces these with user inputs.

Lines 334-336 request EMA20/EMA50 from each MTF with `lookahead=barmerge.lookahead_off`. The engine scores alignment and later applies preset-specific requirements.

MTF pass requirements:

- Strict: 3 of 3
- Balanced: at least 2 of 3
- Aggressive: at least 2 of 3
- Scalping: at least 1 of 3
- Aggressive Scalping: at least 1 of 3

## 14. Entry Conditions

### Long Core

`longCore` at line 850 requires:

- `trendPassLong`
- `mtfPassLong`
- `patternBull`
- `candlePassLong`
- `btcOKLong`
- `fundingOK_long`
- `oiOKLong`
- `sentOK_long`
- `weekendOK`
- no news blackout
- confirmed bar

### Short Core

`shortCore` at line 851 requires:

- `trendPassShort`
- `mtfPassShort`
- `patternBear`
- `candlePassShort`
- `btcOKShort`
- `fundingOK_short`
- `oiOKShort`
- `sentOK_short`
- `weekendOK`
- no news blackout
- confirmed bar

### Final Long Signal

`longSignal` at line 945 requires:

- long trading enabled
- `longCore`
- long confluence count >= `minConfirm`
- reward:risk OK
- confidence >= `minConf`
- cooldown OK
- daily cap OK
- date range OK
- daily drawdown not hit
- HTF support confirmation

### Final Short Signal

`shortSignal` at line 946 mirrors the long signal with short-side conditions and HTF resistance confirmation.

### Entry Orders

Lines 961-976 submit entries only when flat:

- `LONG` for long signals
- `SHORT` for short signals

Quantity is calculated from equity risk divided by stop distance.

## 15. Exit Conditions

### Multi-Target Mode

If `i_useMultiTP` is enabled:

- `TP1 L` / `TP1 S`: exits `i_tp1Pct` percent at TP1
- `TP2 L` / `TP2 S`: exits `i_tp2Pct` percent at TP2
- both exits include the initial stop
- remaining runner is managed by trailing stop logic

### Single-Target Mode

If multi-target mode is disabled:

- `Exit L`: stop at `longSL`, limit at `longTP_simple`
- `Exit S`: stop at `shortSL`, limit at `shortTP_simple`

### Breakeven

Lines 978-999:

- if partial TP mode is enabled and price reaches +1R, a breakeven stop is submitted
- long BE uses `BE L`
- short BE uses `BE S`

### Trailing Stop

Lines 1001-1019:

- long trail: `close - ATR * i_trailATR`, only ratchets upward
- short trail: `close + ATR * i_trailATR`, only ratchets downward
- close below long trail closes `LONG`
- close above short trail closes `SHORT`

### Optional Pyramid Add

Lines 1128-1155:

- if `i_useAccum` is enabled, the strategy adds one `LONG+` or `SHORT+` entry after +1R
- add size is `i_accumPct` percent of original calculated quantity
- flags reset when flat

## 16. Risk Management

Active risk systems:

- equity-percent risk sizing
- ATR-adaptive stop multiplier
- structure-aware stop placement
- minimum reward:risk gate
- multi-target exits
- breakeven stop
- trailing stop
- daily drawdown halt
- daily trade cap
- cooldown
- win/loss streak sizing
- date range filter
- long/short permission switches
- session/weekend/news filters

Stop placement:

- long stop prefers bullish OB bottom, then latest external low, then ATR fallback
- short stop prefers bearish OB top, then latest external high, then ATR fallback
- final stop is bounded against an ATR fallback

Dormant risk systems:

- `equityCurveOK` is calculated but not used
- `maxDDshutdown` is calculated but not used
- `antiRevengeOK` is calculated but not used

## 17. Dashboard

The dashboard is a persistent 2-column, 27-row table.

Creation:

- created once on `barstate.isfirst`

Population:

- updated on `barstate.islast` or `barstate.islastconfirmedhistory`

Rows:

1. Last Signal
2. Preset
3. Wyckoff
4. Chart TF
5. MTF Bias
6. LTF Trend
7. Structure
8. BTC Filter
9. Sentiment
10. HTF S/R
11. RSI
12. MACD
13. Volume
14. CVD
15. Zones
16. Vol Regime
17. Session
18. Funding/News
19. LONG Conf
20. SHORT Conf
21. Confidence
22. Streak
23. Day P&L
24. Backtest
25. Blocker
26. Signal

The blocker row is particularly important: it reports the first failed condition for both long and short branches.

## 18. Alert Engine

Lines 1621-1645 implement dual-format alerts.

Formats:

- plain text
- JSON webhook

Runtime alerts:

- long signals call `alert(longMsg, alert.freq_once_per_bar_close)`
- short signals call `alert(shortMsg, alert.freq_once_per_bar_close)`

Alert conditions:

- long signal
- short signal
- any signal
- daily DD hit

JSON fields:

- `action`
- `symbol`
- `tf`
- `entry`
- `sl`
- `tp1`
- `tp2`
- `rr`
- `confidence`
- `pattern`

## 19. Signal Generation Pipeline

The complete signal path is:

```text
Preset thresholds
  -> EMA trend
  -> MTF trend
  -> external structure
  -> liquidity / OB / FVG / HTF S/R
  -> RSI / MACD / volume / CVD
  -> BTC / sentiment / funding / OI
  -> session / volatility / news / weekend
  -> candle confirmation
  -> pattern qualification
  -> confluence counts
  -> confidence score
  -> stop and target calculation
  -> R:R validation
  -> daily DD / cooldown / daily cap / date range
  -> longSignal or shortSignal
  -> strategy orders
  -> visuals, dashboard, alerts
```

The strategy is confluence-first. A setup must satisfy both a structural/pattern component and enough independent confirmation modules to pass the preset threshold.

## 20. Performance Bottlenecks

### `request.security` Count

The script performs 24 `request.security` calls:

- 3 MTF trend requests
- 2 HTF S/R pivot requests
- 3 BTC trend requests
- 14 sentiment-source requests
- 2 OI requests

This is the largest runtime cost area, especially on low timeframes.

### Visual Object Churn

Several visual components delete and recreate objects:

- OB boxes and labels on last bar
- FVG boxes and labels on last bar
- HTF S/R lines on last bar
- signal banner table while active

The dashboard updates 27 rows on every last-bar refresh. This is acceptable for an overlay strategy, but it is a meaningful rendering workload.

### Long Tooltip Strings

The script builds long explanatory tooltip strings for phase labels, CHOCH labels, OB/FVG labels, signal labels, and dashboard cells. This is user-friendly but contributes to script size and runtime string work.

### Unconditional Calculation

Most engines calculate regardless of whether their master switch is enabled. Examples:

- MTF security requests run even if `i_useMTF=false`
- sentiment security requests run even if `i_useSentiment=false`
- BTC security requests run even if BTC filter is disabled
- OB/FVG engines run regardless of `i_useSMC`
- advanced analytics calculate even though most are not displayed or gated

## 21. Duplicate Calculations

Notable duplicate or consolidatable calculations:

- BTC trend uses three separate security calls for close, EMA20, and EMA50; these could be bundled.
- Sentiment sources use separate close/EMA/RSI security calls per symbol; many can be bundled per symbol.
- OI close and volume are requested separately; they can be bundled.
- Bollinger standard deviation is computed twice at lines 1048 and 1049.
- `ta.sma(volume, 20)` is calculated separately from configurable `volMA`; behavior differs if `i_volLen` is not 20, but the fixed 20 SMA could be stored once.
- `strategy.max_drawdown / strategy.initial_capital * 100` is calculated for analytics and again for dashboard display.
- win rate and profit metrics are calculated in advanced analytics and again in the dashboard.
- Several visual text fragments repeat long/short formatting logic.

## 22. Unused Variables And Functions

There are no user-defined functions in the script.

Variables or inputs that are currently unused or effectively dormant:

| Name | Line | Status |
| --- | ---: | --- |
| `i_useSMC` | 88 | Declared but not used in signal formulas. |
| `i_htfsrLookback` | 145 | Declared but not used by HTF S/R logic. |
| `structOK_long_simple` | 371 | Calculated but unused. |
| `structOK_short_simple` | 372 | Calculated but unused. |
| `btcDom_close` | 549 | Requested but unused. |
| `usdtDom_close` | 554 | Requested but unused. |
| `total_close` | 558 | Requested but unused. |
| `total3_close` | 563 | Requested but unused. |
| `btcDomRSI` | 575 | Calculated but unused. |
| `volShrinking` | 760 | Calculated but not used in accumulation/distribution logic. |
| `phaseFavorsLong` | 797 | Calculated but unused. |
| `phaseFavorsShort` | 798 | Calculated but unused. |
| `zScore` and related variables | 1039-1042 | Calculated but unused. |
| `bbPctB` and related variables | 1044-1050 | Calculated but unused. |
| `marketRegime` | 1062 | Calculated but unused. |
| `expectedValue` | 1073 | Calculated but unused. |
| `kellyHalfFrac` and related variables | 1077-1080 | Calculated but unused. |
| `sortinoApprox` | 1086 | Calculated but unused. |
| `calmarRatio` | 1091 | Calculated but unused. |
| `recoveryFactor` | 1094 | Calculated but unused. |
| `expectancyPer` | 1097 | Calculated but unused. |
| `equityBelowMA` / `equityCurveOK` | 1103-1104 | Calculated but unused. |
| `maxDDshutdown` | 1107-1108 | Calculated but unused. |
| `currentRegime` | 1117 | Calculated but unused. |
| `antiRevengeOK` | 1126 | Calculated but unused. |

Some unused analytics may be intentionally reserved for future dashboard or gating work.

## 23. Areas Where Repainting Could Occur

The script avoids classic future leak by using `lookahead=barmerge.lookahead_off` and confirmed-bar signal gates. Still, there are several repaint-adjacent behaviors to understand.

### Pivot Confirmation Backdating

External pivots use `ta.pivothigh` and `ta.pivotlow` with equal left/right lengths. A pivot is only known after `i_pivotLen` bars have passed. Visual pivot markers use `offset=-i_pivotLen`, so they appear on the historical pivot bar after confirmation.

This is not a direct signal repaint, but visually it can look like the script knew the pivot earlier than it did.

### HTF `request.security` Values

All security calls use `barmerge.lookahead_off`, which prevents future-looking historical values. On realtime bars, higher-timeframe values can still evolve while the higher-timeframe candle is open. This is live-value drift rather than classic lookahead repaint.

Affected modules:

- MTF trend EMAs
- HTF S/R pivots
- BTC trend
- crypto sentiment
- OI proxy

### HTF Pivot Timing

HTF S/R pivots are themselves right-bar-confirmed pivots requested from another timeframe. They appear only after HTF confirmation. This can produce delayed level updates.

### Chart-Bar Confirmation

Most signal gates include `barstate.isconfirmed`, so final signals should not appear before the chart bar closes. However, lower timeframe confirmed bars can still rely on currently developing HTF data from `request.security`.

### Visual-Only Last-Bar Redraw

OB/FVG boxes, HTF lines, the signal banner, and dashboard update on `barstate.islast`. These are visual redraw behaviors, not strategy entry recalculations.

## 24. Areas That Can Be Optimized Without Changing Behavior

The following are behavior-preserving optimization candidates. They should be handled carefully and verified in TradingView.

### Bundle Security Requests

Use tuple requests where the same symbol/timeframe is requested multiple times:

- BTC close, EMA20, EMA50
- OI close and volume
- each sentiment source's close/EMA/RSI fields

This should reduce security call overhead without changing values if expressions are equivalent.

### Remove Or Isolate Unused Calculations

Unused analytics and dormant variables can be removed or placed behind display/gating toggles. Since some may be intended for future features, this should be done only after product confirmation.

### Store Duplicate Statistics

Store repeated calculations once:

- Bollinger standard deviation
- dashboard/backtest max drawdown percent
- dashboard/backtest win rate
- dashboard/backtest profit factor

### Reduce Visual Object Churn

Instead of deleting and recreating some objects on every last-bar update, keep handles and update coordinates/text where Pine APIs allow it.

Likely candidates:

- signal banner table
- HTF S/R lines
- active OB/FVG labels

### Align Switches With Computation

If exact behavior is preserved for enabled states, disabled modules could skip heavy calculation. This is especially relevant for:

- sentiment
- BTC
- OI
- MTF
- advanced analytics

This must be tested carefully because moving `request.security` into conditional branches can affect Pine execution semantics if not done cleanly.

### Consolidate Long/Short Formatting

Alert strings, label tooltips, and dashboard blocker text duplicate long/short formatting. Helper functions or shared string builders could improve maintainability while preserving output.

### Keep Logic And Visuals Separate

Future refactoring can separate:

- pure calculations
- signal gates
- execution
- visuals
- dashboard
- alerts

This would not change behavior by itself, but it would make future quantitative changes safer.

## Key Engineering Conclusions

- The live strategy pipeline is coherent and intentionally confluence-heavy.
- The strongest behavior-critical modules are EMA/MTF trend, pattern qualification, confidence, reward:risk, daily discipline, and HTF S/R.
- The biggest performance pressure is from `request.security` calls and last-bar visual object management.
- The largest maintenance risk is dormant code: several variables and analytics are calculated but not wired into final gates or dashboard output.
- The largest repaint-adjacent risk is pivot confirmation/backdated plotting and evolving HTF data during realtime bars, not explicit lookahead misuse.
- The safest optimization path is documentation-first, then tuple-bundling requests and removing duplicated calculations under hash/backtest comparison.
