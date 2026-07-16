# Module Catalog

This file documents every major module in `crypto-sniper-pro-v5.pine`.

## Preset Selector

Lines: 61-78

Purpose: converts a user-facing preset into threshold values used by the signal engine.

Inputs:

- `i_preset`
- custom override inputs when preset is Custom

Outputs:

- `minConfirm`
- `minConf`
- `cooldown`
- `minRR`
- `maxPerDay`
- `isScalping`
- `isAggScalp`

Consumers:

- signal engine
- risk checks
- cooldown checks
- daily cap checks
- trend and candle pass logic

## Master Switches

Lines: 84-104

Purpose: enable or disable major filters and execution features.

Switches:

- MTF trend stack
- external structure
- smart money zones
- BTC correlation
- HTF S/R
- RSI
- MACD
- volume
- session
- volatility regime
- CVD
- funding
- OI
- news blackout
- crypto sentiment
- partial TP and BE
- multi-target TP
- daily loss circuit breaker
- streak sizing

Observed note: `i_useSMC` is declared but not used by the current signal formulas.

## Adaptive Timeframe Resolution

Lines: 293-305

Purpose: chooses three higher timeframes based on the chart timeframe unless manual MTF mode is selected.

Outputs:

- `mtf1`
- `mtf2`
- `mtf3`

Consumers:

- MTF trend stack
- dashboard Chart TF row

## ATR And Volatility State

Lines: 310-314, 658-672

Purpose: establishes volatility context for stops, filters, sweeps, zones, and visuals.

Outputs:

- `atr`
- `atrLong`
- `atrRatio`
- `isLowVol`
- `isHighVol`
- `atrPct`
- `adx`
- `volRegimeOK`

Consumers:

- liquidity sweep threshold
- OB and FVG thresholds
- HTF S/R proximity
- stop placement
- Wyckoff phase logic
- confluence counts
- dashboard

## EMA Engine

Lines: 318-331

Purpose: creates lower-timeframe trend state from EMA20, EMA50, EMA200, and EMA50 slope.

Inputs:

- chart close

Outputs:

- `ema20`
- `ema50`
- `ema200`
- `emaBull`
- `emaBear`
- `emaSlope`
- `ltfTrendBull`
- `ltfTrendBear`
- `trendLabel`

Consumers:

- trend pass logic
- Wyckoff markup/markdown
- dashboard
- visual EMA plots

## MTF Trend Engine

Lines: 333-346, 837-839

Purpose: checks higher-timeframe EMA20/EMA50 alignment.

Inputs:

- `mtf1`, `mtf2`, `mtf3`
- `i_useMTF`
- preset flags

Outputs:

- `mtf1Bias`
- `mtf2Bias`
- `mtf3Bias`
- `mtfBullScore`
- `mtfBearScore`
- `mtfFullBull`
- `mtfFullBear`
- `mtfPassLong`
- `mtfPassShort`

Consumers:

- core signal gates
- confidence score
- dashboard

## External Structure Engine

Lines: 348-380

Purpose: detects major external swing structure using pivot highs and lows.

Inputs:

- `i_pivotLen`
- `i_bosMinATR`

Outputs:

- `lastExtHigh`, `prevExtHigh`, `lastExtHighBar`
- `lastExtLow`, `prevExtLow`, `lastExtLowBar`
- `extBullStruct`
- `extBearStruct`
- `structureBias`
- `bosBull`
- `bosBear`
- `chochBull`
- `chochBear`

Consumers:

- confluence counts
- breakout retest patterns
- liquidity sweeps
- stop placement
- dashboard
- structure markers and CHoCH labels

Observed note: `structOK_long_simple` and `structOK_short_simple` are calculated but not consumed.

## Liquidity Engine

Lines: 382-394

Purpose: identifies high and low sweeps near external pivots.

Inputs:

- `i_liqNearATR`
- `i_requireTrend`
- latest external pivot levels
- ATR

Outputs:

- `sweepHigh`
- `sweepLow`

Consumers:

- sweep reversal pattern
- Wyckoff manipulation phase
- sweep markers

## Order Block Engine

Lines: 396-430

Purpose: tracks the latest fresh bullish and bearish order blocks created by impulse candles.

Inputs:

- `i_obImpulse`
- `i_obMaxBars`
- ATR

Outputs:

- `bullOB_top`
- `bullOB_bot`
- `bullOB_bar`
- `bullOBValid`
- `bearOB_top`
- `bearOB_bot`
- `bearOB_bar`
- `bearOBValid`
- `inBullOB`
- `inBearOB`

Consumers:

- pattern engine
- stop placement
- dashboard zones
- OB visualization

## Fair Value Gap Engine

Lines: 432-458

Purpose: tracks active bullish and bearish three-candle imbalances.

Inputs:

- `i_fvgMinATR`
- ATR

Outputs:

- `bullFVG`
- `bearFVG`
- `bullFVG_top`
- `bullFVG_bot`
- `bullFVG_active`
- `bearFVG_top`
- `bearFVG_bot`
- `bearFVG_active`
- `inBullFVG`
- `inBearFVG`

Consumers:

- pattern engine
- dashboard zones
- FVG visualization

## HTF Support/Resistance Engine

Lines: 460-480

Purpose: stores recent higher-timeframe pivot levels and checks proximity.

Inputs:

- `i_htfsrTF`
- `i_htfsrPivotLen`
- `i_htfsrProxATR`

Outputs:

- `htfR1`, `htfR2`
- `htfS1`, `htfS2`
- `nearHTFsupport`
- `nearHTFresist`
- `htfsrConfLong`
- `htfsrConfShort`

Consumers:

- confluence count
- confidence score
- final signal gate
- dashboard
- HTF S/R lines

Observed note: `i_htfsrLookback` is declared but not used.

## RSI Engine

Lines: 482-499

Purpose: provides RSI zone and divergence confirmation.

Inputs:

- `i_rsiLen`
- `i_rsiOB`
- `i_rsiOS`
- `i_rsiDivLen`

Outputs:

- `rsi`
- `rsiBullZone`
- `rsiBearZone`
- `rsiOversold`
- `rsiOverbought`
- `bullDiv`
- `bearDiv`
- `rsiConfBull`
- `rsiConfBear`

Consumers:

- confluence counts
- dashboard

## MACD Engine

Lines: 501-509

Purpose: provides momentum shift and state confirmation.

Inputs:

- `i_macdFast`
- `i_macdSlow`
- `i_macdSig`

Outputs:

- `macdBullShift`
- `macdBearShift`
- `macdBullState`
- `macdBearState`
- `macdConfBull`
- `macdConfBear`

Consumers:

- confluence counts
- dashboard

## Volume Engine

Lines: 511-514

Purpose: validates moves with a volume spike.

Inputs:

- `i_volLen`
- `i_volSpike`

Outputs:

- `volMA`
- `volSpike`
- `volConfirm`

Consumers:

- confluence counts
- Wyckoff manipulation
- dashboard

## CVD Engine

Lines: 516-531

Purpose: estimates buy/sell pressure and CVD divergence from candle-direction volume.

Inputs:

- `i_cvdLen`
- `i_cvdDivLen`

Outputs:

- `cvdDelta`
- `cvdBullDiv`
- `cvdBearDiv`
- `cvdConfBull`
- `cvdConfBear`

Consumers:

- confluence counts
- dashboard

## BTC Correlation Engine

Lines: 533-541

Purpose: requires non-BTC charts to align with BTC trend when enabled.

Inputs:

- `i_btcSymbol`
- `i_btcTrendTF`
- `i_useBTC`

Outputs:

- `btcBull`
- `btcBear`
- `btcOKLong`
- `btcOKShort`

Consumers:

- core signal gates
- dashboard
- signal tooltips

## Crypto Sentiment Engine

Lines: 543-628

Purpose: builds a crypto sentiment score from dominance and market-cap sources.

Inputs:

- `i_btcDomSymbol`
- `i_usdtDomSymbol`
- `i_totalMcSymbol`
- `i_total3Symbol`
- `i_sentTF`
- `i_sentExtreme`

Outputs:

- `sentimentScore`
- `sentimentLabel`
- `extremeGreed`
- `extremeFear`
- `sentOK_long`
- `sentOK_short`
- `sentBoostsLong`
- `sentBoostsShort`

Consumers:

- core signal gates
- confidence score
- dashboard

## Funding Gate

Lines: 630-637

Purpose: manual contrarian crowding filter.

Inputs:

- `i_fundingPct`
- `i_useFunding`

Outputs:

- `fundingOK_long`
- `fundingOK_short`

Consumers:

- core signal gates
- dashboard

## Open Interest Gate

Lines: 639-650

Purpose: validates directional price moves using OI or a volume-price proxy.

Inputs:

- `i_oiSymbol`
- `i_oiUseProxy`
- `i_useOI`

Outputs:

- `oiOKLong`
- `oiOKShort`

Consumers:

- core signal gates

## Session Engine

Lines: 652-656

Purpose: gates trades to London or NY sessions and optionally skips weekends.

Inputs:

- `i_sessLondon`
- `i_sessNY`
- `i_sessTZ`
- `i_skipWeekend`

Outputs:

- `inSession`
- `weekendOK`

Consumers:

- confluence counts
- core signal gates
- dashboard

## News Blackout Engine

Lines: 674-718

Purpose: blocks trading around recurring or manual event windows.

Inputs:

- auto news controls
- FOMC/CPI/NFP toggles
- daily reset toggle
- weekend open toggle
- manual event windows

Outputs:

- `autoBlackout`
- `manualBlackout`
- `inNewsBlackout`
- `newsEventName`

Consumers:

- core signal gates
- dashboard
- background tint

## Candle Confirmation Engine

Lines: 719-725, 845-847

Purpose: determines bullish or bearish candle confirmation and relaxes it for scalping presets.

Outputs:

- `bullEngulf`
- `bearEngulf`
- `bullClose`
- `bearClose`
- `candleBull`
- `candleBear`
- `candlePassLong`
- `candlePassShort`

Consumers:

- pattern engine
- core signal gates
- confidence score

## Pattern Engine

Lines: 727-737

Purpose: requires an actionable setup pattern before entries.

Bullish patterns:

- breakout retest above previous external high
- sweep low reversal
- bullish OB tap
- bullish FVG fill

Bearish patterns:

- breakout retest below previous external low
- sweep high reversal
- bearish OB tap
- bearish FVG fill

Outputs:

- `patternBull`
- `patternBear`
- `patternReason`

Consumers:

- core signal gates
- confidence score
- alerts
- signal labels

## Wyckoff Phase Engine

Lines: 739-798

Purpose: classifies current phase into accumulation, manipulation, markup, distribution, markdown, or neutral.

Inputs:

- range state
- ATR contraction
- volume state
- prior trend
- liquidity sweeps
- EMA trend

Outputs:

- `currentPhase`
- `phaseColor`
- `phaseFavorsLong`
- `phaseFavorsShort`

Consumers:

- background tints
- phase labels
- dashboard

Observed note: `phaseFavorsLong` and `phaseFavorsShort` are calculated but not used in signal gates or confidence.

## Confluence And Confidence Engine

Lines: 800-861

Purpose: counts enabled confirmations and creates weighted confidence scores.

Outputs:

- `longConfCount`
- `shortConfCount`
- `activeModules`
- `confLong`
- `confShort`

Consumers:

- final signal gates
- dashboard
- alerts
- signal labels

## Risk Calculation Engine

Lines: 863-894, 955-958

Purpose: defines stops, targets, R:R, and position quantity.

Inputs:

- risk percent
- ATR stop settings
- OB/structure levels
- TP inputs
- streak multiplier

Outputs:

- `longSL`
- `shortSL`
- `longTP1`, `longTP2`
- `shortTP1`, `shortTP2`
- `longTP_simple`
- `shortTP_simple`
- `longRR`
- `shortRR`
- `rrOK_long`
- `rrOK_short`
- `riskAmount`
- `qtyLong`
- `qtyShort`

Consumers:

- final signal gates
- strategy orders
- chart plots
- dashboard and alerts

## Discipline Engine

Lines: 896-950

Purpose: manages day-level and sequence-level trade limits.

Inputs:

- `i_useDailyDD`
- `i_maxDailyDD`
- streak settings
- preset cooldown and max-per-day
- backtest date range

Outputs:

- `dailyDD_pct`
- `dailyDD_hit`
- `winStreakC`
- `lossStreakC`
- `streakMult`
- `cooldownOK`
- `dailyCapOK`
- `inDateRange`
- `longSignal`
- `shortSignal`

Consumers:

- entries
- dashboard
- alerts
- background tints

## Strategy Execution

Lines: 952-1019

Purpose: turns final signals into orders and manages exits.

Entry IDs:

- `LONG`
- `SHORT`

Exit IDs:

- `TP1 L`, `TP2 L`, `Exit L`, `BE L`
- `TP1 S`, `TP2 S`, `Exit S`, `BE S`

Close comments:

- `Trail`

## Advanced Analytics Engine

Lines: 1021-1126

Purpose: calculates performance and statistical diagnostics.

Outputs include:

- `sharpeApprox`
- `zScore`
- `bbPctB`
- `hurstApprox`
- `marketRegime`
- `expectedValue`
- `kellyHalfFrac`
- `sortinoApprox`
- `calmarRatio`
- `recoveryFactor`
- `expectancyPer`
- `equityCurveOK`
- `maxDDshutdown`
- `currentRegime`
- `antiRevengeOK`

Observed note: most analytics are not wired into the final signal gates. `sharpeApprox` is displayed in the dashboard.

## Accumulation Pyramid Module

Lines: 1128-1155

Purpose: optional add-on entry at +1R.

Inputs:

- `i_useAccum`
- `i_accumPct`

Outputs:

- `LONG+` or `SHORT+` entries when enabled and conditions are met.

Consumers:

- strategy execution only.

## Visualization Module

Lines: 1157-1394

Purpose: renders phase tints, labels, EMAs, structure, zones, signal markers, and SL/TP/trailing plots.

Controls:

- `i_showEMA`
- `i_showStruct`
- `i_showOB`
- `i_showFVG`
- `i_showHTFsr`
- `i_showWyckoff`
- `i_showPhaseLbl`
- `i_showSignals`
- `i_showSLTP`

## Signal Banner

Lines: 1390-1452

Purpose: captures the last signal and displays a centered table for 50 bars or while a position is open.

Fields:

- direction
- ticker and timeframe
- entry
- stop
- TP1
- TP2
- R:R
- confidence
- position status

## Dashboard

Lines: 1454-1619

Purpose: persistent status panel with trade, trend, confluence, risk, and performance diagnostics.

Consumers:

- nearly every upstream module.

## Alerts

Lines: 1621-1645

Purpose: emits signal alerts in plain text or JSON format and exposes alert conditions.

Alert paths:

- runtime `alert()` on long/short signal
- `alertcondition()` for long, short, any signal, and daily DD hit
