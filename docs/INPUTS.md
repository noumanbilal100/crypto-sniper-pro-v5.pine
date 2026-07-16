# Inputs Reference

This document lists the input groups in `crypto-sniper-pro-v5.pine`.

## Preset

- `i_preset`: selects Strict, Balanced, Aggressive, Scalping, Aggressive Scalping, or Custom.

## Master Switches

- `i_useMTF`: MTF trend stack.
- `i_useStruct`: external structure.
- `i_useSMC`: smart money zones input, currently not wired into signal formulas.
- `i_useBTC`: BTC correlation gate.
- `i_useHTFsr`: HTF S/R confluence.
- `i_useRSI`: RSI confirmation.
- `i_useMACD`: MACD confirmation.
- `i_useVol`: volume validation.
- `i_useSession`: session filter.
- `i_useVolReg`: volatility regime.
- `i_useCVD`: CVD divergence.
- `i_useFunding`: funding rate gate.
- `i_useOI`: open interest divergence.
- `i_useNewsBlock`: news/event blackout.
- `i_useSentiment`: crypto sentiment engine.
- `i_usePartialTP`: partial TP and breakeven.
- `i_useMultiTP`: multi-target TP.
- `i_useDailyDD`: daily loss circuit breaker.
- `i_useStreakAdj`: win/loss streak sizing.

## Custom Overrides

Used only when preset is Custom:

- `i_customMinConf`
- `i_customConfPct`
- `i_customCooldown`
- `i_customRR`
- `i_customMaxDay`

## Adaptive MTF Stack

- `i_mtfMode`
- `i_mtf1`
- `i_mtf2`
- `i_mtf3`

## Structure / SMC

- `i_pivotLen`
- `i_bosMinATR`
- `i_obImpulse`
- `i_obMaxBars`
- `i_fvgMinATR`

## Liquidity Filter

- `i_liqNearATR`
- `i_requireTrend`

## HTF S/R Confluence

- `i_htfsrTF`
- `i_htfsrLookback`: declared but not used in current logic.
- `i_htfsrPivotLen`
- `i_htfsrProxATR`

## RSI

- `i_rsiLen`
- `i_rsiOB`
- `i_rsiOS`
- `i_rsiDivLen`

## MACD

- `i_macdFast`
- `i_macdSlow`
- `i_macdSig`

## Volume + CVD

- `i_volLen`
- `i_volSpike`
- `i_cvdLen`
- `i_cvdDivLen`

## BTC Correlation

- `i_btcSymbol`
- `i_btcTrendTF`

## Funding & OI

- `i_fundingPct`
- `i_oiSymbol`
- `i_oiUseProxy`

## Crypto Sentiment

- `i_btcDomSymbol`
- `i_usdtDomSymbol`
- `i_totalMcSymbol`
- `i_total3Symbol`
- `i_sentTF`
- `i_sentExtreme`

## Session Filter

- `i_sessLondon`
- `i_sessNY`
- `i_sessTZ`
- `i_skipWeekend`

## Volatility Regime

- `i_atrMinPct`
- `i_adxLen`
- `i_adxMin`

## News / Event Blackout

- `i_newsAuto`
- `i_newsAutoBefore`
- `i_newsAutoAfter`
- `i_newsBlockFOMC`
- `i_newsBlockCPI`
- `i_newsBlockNFP`
- `i_newsBlockMidnight`
- `i_newsBlockWeekend`
- `i_news1Start`
- `i_news1End`
- `i_news2Start`
- `i_news2End`
- `i_news3Start`
- `i_news3End`

## Risk Management

- `i_riskPct`
- `i_slBufATR`
- `i_adaptiveATR`
- `i_atrLowMult`
- `i_atrHighMult`

## Multi-Target TP

- `i_tp1R`
- `i_tp1Pct`
- `i_tp2R`
- `i_tp2Pct`
- `i_trailATR`

## Daily Drawdown

- `i_maxDailyDD`

## Streak Sizing

- `i_lossStreak`
- `i_lossStreakReduce`
- `i_winStreak`
- `i_winStreakAdd`

## Discipline

- `i_allowLong`
- `i_allowShort`

## Backtest Range

- `i_useDate`
- `i_dateStart`
- `i_dateEnd`

## Visualization

- `i_showEMA`
- `i_showStruct`
- `i_showOB`
- `i_showFVG`
- `i_showHTFsr`
- `i_showWyckoff`
- `i_showPhaseLbl`
- `i_showSignals`
- `i_showSLTP`
- `i_showDash`
- `i_dashPos`
- `i_textSize`
- `i_labelSize`

## Alerts

- `i_alertFormat`

## Accumulation Strategy

- `i_useAccum`
- `i_accumPct`
