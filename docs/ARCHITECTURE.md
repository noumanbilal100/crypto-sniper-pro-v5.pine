# Architecture Report

## Scope

This report documents the architecture of `crypto-sniper-pro-v5.pine` exactly as implemented. No trading logic has been changed.

The script is a monolithic Pine Strategy organized by large comment sections. The effective architecture is a pipeline:

```text
Inputs and presets
  -> timeframe resolution
  -> market context engines
  -> pattern and phase engines
  -> confluence counts and confidence score
  -> final signal gates
  -> strategy execution
  -> analytics
  -> visuals, dashboard, alerts
```

## Source Layout

| Lines | Area |
| ---: | --- |
| 1-59 | Header and `strategy()` declaration |
| 61-119 | Presets, master switches, custom thresholds |
| 121-291 | Input groups |
| 293-305 | Adaptive timeframe resolution |
| 307-542 | Core market engines: ATR, EMA, MTF, structure, liquidity, OB, FVG, HTF S/R, RSI, MACD, volume, CVD, BTC |
| 543-650 | Crypto sentiment, funding, OI |
| 652-718 | Session, volatility regime, news blackout |
| 719-798 | Candle, pattern, Wyckoff phase |
| 800-894 | Confluence counts, adaptive signal requirements, confidence, risk targets |
| 896-950 | Discipline, daily DD, streaks, cooldown, final signals |
| 952-1030 | Strategy entries, exits, breakeven, trailing stop, Sharpe approximation |
| 1032-1155 | Advanced analytics and optional accumulation pyramid |
| 1157-1452 | Visual components and giant signal banner |
| 1454-1619 | Dashboard |
| 1621-1645 | Alerts |

## Strategy Declaration

The strategy is declared as an overlay strategy with:

- `initial_capital=10000`
- `default_qty_type=strategy.percent_of_equity`
- `default_qty_value=10`
- `commission_type=strategy.commission.percent`
- `commission_value=0.05`
- `slippage=2`
- `pyramiding=1`
- `calc_on_every_tick=false`
- `process_orders_on_close=true`
- object limits for boxes, lines, and labels set to 150
- `max_bars_back=500`

Because `calc_on_every_tick=false` and most signal components require `barstate.isconfirmed`, the system is built around confirmed-bar decisions, not intrabar scalping decisions.

## Input Architecture

The input layer has four types of controls:

1. Preset controls: define global strictness.
2. Master switches: enable or disable groups of filters.
3. Module parameters: tune each engine.
4. Output controls: show/hide visuals, dashboard, alerts.

Preset-derived values:

| Value | Purpose |
| --- | --- |
| `minConfirm` | Required count of enabled confluence modules. |
| `minConf` | Required weighted confidence score. |
| `cooldown` | Minimum bars between signals. |
| `minRR` | Minimum reward:risk requirement. |
| `maxPerDay` | Daily trade cap. |

The Custom preset replaces all five values with user inputs.

## Data Flow

```text
Price, volume, time, symbol
  -> ATR and volatility state
  -> EMA engine
     -> LTF trend
     -> MTF trend stack
  -> structure engine
     -> external pivots
     -> BOS and CHoCH
     -> liquidity sweeps
     -> order blocks
     -> fair value gaps
     -> HTF support/resistance
  -> indicator confirmations
     -> RSI, MACD, volume, CVD
  -> market filters
     -> BTC trend, sentiment, funding, OI, session, weekend, news, volatility
  -> pattern engine
     -> breakout retest, sweep reversal, OB tap, FVG fill
  -> Wyckoff phase engine
  -> confluence and confidence engine
  -> risk engine
     -> SL, TP1, TP2, simple TP, R:R, quantity
  -> final signal gate
  -> strategy.entry / strategy.exit / strategy.close
  -> visuals / dashboard / alerts
```

## Signal Engine

The signal engine begins after all market context and filters are calculated.

### Confluence Inputs

Long confluence modules:

- RSI bullish confirmation
- MACD bullish confirmation
- Volume confirmation
- Non-bearish structure bias
- Active session
- CVD bullish confirmation
- HTF support confluence
- Volatility regime OK

Short confluence modules:

- RSI bearish confirmation
- MACD bearish confirmation
- Volume confirmation
- Non-bullish structure bias
- Active session
- CVD bearish confirmation
- HTF resistance confluence
- Volatility regime OK

Disabled modules are not counted as passing confirmations because each count item is gated as `i_useX and condition`.

### Core Long Requirement

`longCore` requires:

- trend pass
- MTF pass
- bullish pattern
- bullish candle pass
- BTC long filter
- funding long filter
- OI long filter
- sentiment long filter
- weekend filter
- no news blackout
- confirmed bar

### Core Short Requirement

`shortCore` mirrors long logic with bearish trend, bearish MTF, bearish pattern, bearish candle, BTC short filter, funding short filter, OI short filter, sentiment short filter, weekend filter, no news blackout, and confirmed bar.

### Final Signal Gates

`longSignal` requires:

- `i_allowLong`
- `longCore`
- `longConfCount >= minConfirm`
- `rrOK_long`
- `confLong >= minConf`
- `cooldownOK`
- `dailyCapOK`
- `inDateRange`
- not `dailyDD_hit`
- `htfsrConfLong`

`shortSignal` is the bearish equivalent.

The HTF S/R requirement appears twice when enabled: inside confluence/confidence and again as a final signal gate.

## Trend Engine

The trend engine is a composite of:

- LTF EMA stack and slope.
- MTF EMA bias stack.
- External structure bias.
- Volatility regime via ATR% and ADX.
- BTC correlation trend for non-BTC charts.

The final trend pass used by entries depends on preset:

- Strict and Balanced style use strong LTF trend: EMA stack plus EMA50 slope.
- Aggressive and scalping styles accept EMA stack without slope.
- MTF requirements relax as presets become more aggressive.

## EMA Engine

The EMA engine calculates:

- `ema20`
- `ema50`
- `ema200`
- `emaBull`: `ema20 > ema50 > ema200`
- `emaBear`: `ema20 < ema50 < ema200`
- `emaSlope`: five-bar change in EMA50
- `ltfTrendBull`: bullish EMA stack and rising EMA50
- `ltfTrendBear`: bearish EMA stack and falling EMA50
- `trendLabel`: dashboard and tooltip text.

The EMA engine feeds:

- LTF trend gate.
- Wyckoff markup/markdown detection.
- dashboard LTF trend row.
- chart EMA plots.

## MTF Trend Stack

The MTF engine auto-resolves three higher timeframes from the chart timeframe unless Manual mode is selected.

For each MTF, the script pulls EMA20 and EMA50 using `request.security(..., lookahead=barmerge.lookahead_off)`.

Bias values:

- `1`: EMA20 above EMA50
- `-1`: EMA20 below EMA50
- `0`: neutral

Scores:

- `mtfBullScore`: count of bullish MTFs.
- `mtfBearScore`: count of bearish MTFs.
- `mtfFullBull`: all three bullish or MTF disabled.
- `mtfFullBear`: all three bearish or MTF disabled.

Preset requirements:

- Strict: 3 of 3 MTFs.
- Balanced: at least 2 of 3.
- Aggressive: at least 2 of 3.
- Scalping: at least 1 of 3.
- Aggressive Scalping: at least 1 of 3.

## Smart Money Engine

The smart money layer includes:

- External market structure pivots.
- BOS and CHoCH.
- Liquidity sweeps.
- Order blocks.
- Fair value gaps.
- HTF support/resistance.
- Pattern engine.
- Wyckoff phase engine.

These modules are intertwined rather than isolated. The pattern engine is the main consumer:

- breakout retest
- sweep reversal
- active OB interaction
- active FVG interaction

Observed behavior: the `i_useSMC` master switch is declared but not referenced in the current signal formulas. OB/FVG can still influence `patternBull` and `patternBear` even if that switch is turned off.

## Liquidity Engine

The liquidity engine detects stop-hunt style sweeps at external pivots.

Inputs:

- `i_liqNearATR`: maximum ATR distance from pivot.
- `i_requireTrend`: whether a prior move is required.

Logic:

- Prior move is `abs(close - close[10])`.
- A meaningful prior trend move is greater than `atr * 3`.
- High sweep: high trades above last external high and closes back below it.
- Low sweep: low trades below last external low and closes back above it.
- Sweep must be near the pivot and confirmed.

Consumers:

- sweep reversal pattern
- Wyckoff manipulation phase
- chart sweep markers

## Order Block Engine

The order block engine identifies the previous opposite-colored candle before a large impulse.

Bullish OB:

- current candle is a bullish impulse greater than `atr * i_obImpulse`
- previous candle was bearish
- OB top is previous open
- OB bottom is previous close

Bearish OB:

- current candle is a bearish impulse greater than `atr * i_obImpulse`
- previous candle was bullish
- OB top is previous close
- OB bottom is previous open

Freshness:

- A bullish OB becomes invalid when price trades to or below its bottom.
- A bearish OB becomes invalid when price trades to or above its top.
- OBs expire after `i_obMaxBars`.

Consumers:

- `inBullOB` / `inBearOB`
- pattern engine
- structure-based stop placement
- dashboard zones row
- OB boxes and labels

## Fair Value Gap Engine

The FVG engine detects three-candle imbalances:

- Bullish FVG: current low is above the high two bars ago, and gap size is at least `atr * i_fvgMinATR`.
- Bearish FVG: current high is below the low two bars ago, and gap size is at least `atr * i_fvgMinATR`.

State:

- latest bullish and bearish gap boundaries are stored.
- bullish gap deactivates when low reaches the gap bottom.
- bearish gap deactivates when high reaches the gap top.

Consumers:

- `inBullFVG` / `inBearFVG`
- pattern engine
- dashboard zones row
- FVG boxes and labels

## HTF Support/Resistance Engine

The HTF S/R engine pulls pivot highs and lows from `i_htfsrTF`, stores the two latest resistance levels and two latest support levels, then checks proximity by ATR.

Outputs:

- `nearHTFsupport`
- `nearHTFresist`
- `htfsrConfLong`
- `htfsrConfShort`

Consumers:

- confluence counts
- confidence score
- final signal gates
- dashboard row
- chart lines

Observed behavior: `i_htfsrLookback` is declared but not used in the pivot storage or proximity logic.

## Momentum And Confirmation Engines

RSI:

- bullish zone: RSI between 45 and 70
- bearish zone: RSI between 30 and 55
- oversold and overbought thresholds from inputs
- divergence is based on price lower-low or higher-high against RSI behavior

MACD:

- bullish shift: crossover or improving histogram below zero
- bearish shift: crossunder or weakening histogram above zero
- state checks require MACD line and histogram alignment

Volume:

- spike when current volume exceeds SMA volume times threshold
- confirmation requires spike and confirmed bar when enabled

CVD:

- proxy uses bullish candle volume minus bearish candle volume, smoothed with EMA
- bullish/bearish divergence compares price extremes against CVD extremes

## BTC, Sentiment, Funding, And OI

BTC filter:

- pulls BTC close, EMA20, and EMA50 from configured symbol and timeframe.
- non-BTC charts require BTC bull trend for longs and BTC bear trend for shorts when enabled.
- charts whose ticker contains `BTC` bypass the BTC filter.

Sentiment:

- pulls BTC dominance, USDT dominance, TOTAL, and TOTAL3 data.
- builds a 0-100 score from TOTAL trend, stablecoin dominance trend, TOTAL RSI, TOTAL3 trend, and BTC dominance behavior.
- extreme greed blocks longs.
- extreme fear blocks shorts.
- aligned sentiment adds confidence.

Funding:

- manual funding input.
- funding greater than 0.05 blocks longs when enabled.
- funding less than -0.03 blocks shorts when enabled.

OI:

- uses configured symbol close and volume.
- proxy is either `close * volume` or volume.
- if price rises, OI must rise for longs.
- if price falls, OI must rise for shorts.

## Session, Volatility, And News

Session:

- London and NY sessions are input sessions.
- `inSession` passes if either session is active or session filter is disabled.
- weekend skip is separate from session filter.

Volatility:

- ATR percent must exceed `i_atrMinPct`.
- manually implemented ADX must exceed `i_adxMin`.

News blackout:

- recurring windows for FOMC, CPI, NFP, optional daily reset, and optional weekend open.
- three manual event windows.
- if enabled, blackout blocks `longCore` and `shortCore`.

## Entry Logic

Entry conditions:

- `longSignal` or `shortSignal` must be true.
- current position must be flat: `strategy.position_size == 0`.

Orders:

- Long entry ID: `LONG`.
- Short entry ID: `SHORT`.
- Quantity is risk-based: account equity times risk percent times streak multiplier divided by stop distance.

Optional accumulation entries:

- If enabled, `LONG+` or `SHORT+` is attempted when an open trade reaches +1R.
- Add size is a percentage of the original calculated quantity.

## Exit Logic

Multi-target mode:

- TP1 exits `i_tp1Pct` percent at `i_tp1R` multiples of risk.
- TP2 exits `i_tp2Pct` percent at `i_tp2R` multiples of risk.
- Both exits include the initial stop.
- Runner is handled by trailing stop logic.

Non-multi-target mode:

- Single exit uses structure/target-derived `longTP_simple` or `shortTP_simple`.

Partial TP and breakeven:

- When enabled, once price reaches +1R, a breakeven stop is submitted for the remaining position.

Trailing stop:

- Long trail moves upward with `close - atr * i_trailATR`.
- Short trail moves downward with `close + atr * i_trailATR`.
- If close crosses the trail against the position, the strategy closes the position.

## Risk Management

Risk components:

- ATR-based stop multiplier.
- Structure-aware stop placement.
- Reward:risk validation.
- Equity-based position sizing.
- Daily drawdown circuit breaker.
- Daily trade cap.
- Cooldown between signals.
- Win/loss streak size multiplier.
- Backtest date filter.
- Long/short permission switches.

Stop logic:

- Long stop prefers bullish OB bottom or last external low, with ATR buffer, but is bounded against an ATR fallback.
- Short stop prefers bearish OB top or last external high, with ATR buffer, but is bounded against an ATR fallback.

Dormant calculated risk components:

- `equityCurveOK`
- `maxDDshutdown`
- `antiRevengeOK`

These are calculated but not included in final signal gates in the current source.

## Dashboard

The dashboard is a 2-column, 27-row table created on the first bar and populated on the last bar or last confirmed history bar.

Rows include:

- Last Signal
- Preset
- Wyckoff
- Chart TF
- MTF Bias
- LTF Trend
- Structure
- BTC Filter
- Sentiment
- HTF S/R
- RSI
- MACD
- Volume
- CVD
- Zones
- Vol Regime
- Session
- Funding/News
- Long confirmations
- Short confirmations
- Confidence
- Streak
- Day P&L
- Backtest
- Blocker diagnostic
- Signal

The blocker row reports the first failed condition for long and short paths.

## Alerts

Alert architecture:

- Plain text message for long and short.
- JSON webhook message for long and short.
- User input selects format.
- Runtime `alert()` fires once per bar close on signal.
- `alertcondition()` supports LONG, SHORT, ANY, and daily DD hit.

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

## Visual Components

Visuals include:

- Wyckoff phase background tints.
- Phase transition labels with tooltips.
- EMA plots.
- External pivot markers.
- Latest external high/low dashed lines.
- CHoCH labels.
- Liquidity sweep markers.
- OB boxes and labels.
- FVG boxes and labels.
- HTF S/R lines.
- Long/short signal labels.
- Signal bar coloring and signal background flashes.
- Daily DD and news blackout background tints.
- SL, TP1, TP2, and trailing stop plots.
- Giant signal banner table.
- Dashboard table.

## Analytics Engine

The analytics engine calculates:

- Sharpe approximation.
- Z-score.
- Bollinger %B.
- Hurst proxy and market regime label.
- live win rate, average win, average loss.
- expected value.
- Kelly and half-Kelly fractions.
- Sortino approximation.
- Calmar ratio.
- recovery factor.
- expectancy per trade.
- equity curve MA state.
- hardcoded 15% max DD shutdown flag.
- current market regime.
- anti-revenge cooldown flag.

Only Sharpe approximation is displayed in the main dashboard performance row. The other analytics are calculated but not connected to final entry gates in the current script.

## Observed Implementation Notes

- `i_useSMC` is input-only in the current source.
- `i_htfsrLookback` is input-only in the current source.
- `structOK_long_simple` and `structOK_short_simple` are calculated but not used.
- `btcDomRSI` is calculated but not used.
- `volShrinking` is calculated but not used by accumulation/distribution.
- Advanced analytics are mostly calculated-only.
- `equityCurveOK`, `maxDDshutdown`, and `antiRevengeOK` are not wired into `longSignal` or `shortSignal`.
- The dashboard tooltip says MTF must fully agree, but actual MTF requirements are preset-adaptive.
- The confidence comment and dashboard tooltip do not exactly match the implemented arithmetic. Implemented confidence uses MTF 25 or 8 per aligned MTF, pattern 15, candle 10, confluence count times 3, HTF S/R 10, and sentiment 5 or 10.
