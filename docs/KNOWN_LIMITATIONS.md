# Known Limitations

This document records observed behavior in the current source. It is not a bug-fix list and does not change strategy logic.

## Master Switch Wiring

`i_useSMC` is declared as `Smart Money Zones (OB/FVG)`, but current signal formulas do not reference it.

Impact:

- OB/FVG calculations still run.
- `inBullOB`, `inBearOB`, `inBullFVG`, and `inBearFVG` can still feed `patternBull` and `patternBear`.
- Visual OB/FVG controls remain separate through `i_showOB` and `i_showFVG`.

## HTF S/R Lookback

`i_htfsrLookback` is declared but not used by the current HTF S/R calculation.

Impact:

- HTF pivots are stored as latest two support and resistance values.
- The configured lookback does not currently limit or expand HTF level selection.

## Dormant Risk Controls

The following are calculated but not wired into `longSignal` or `shortSignal`:

- `equityCurveOK`
- `maxDDshutdown`
- `antiRevengeOK`

Impact:

- Daily drawdown, cooldown, daily trade cap, and streak sizing are active.
- Equity-curve shutdown, hard max-DD shutdown, and anti-revenge blocking are not active final gates.

## Dormant Analytics

The following values are calculated but not displayed or used by entry gates:

- `zScore`
- `bbPctB`
- `marketRegime`
- `currentRegime`
- `expectedValue`
- `kellyHalfFrac`
- `sortinoApprox`
- `calmarRatio`
- `recoveryFactor`
- `expectancyPer`

## Dashboard Text Drift

Some comments or dashboard tooltips describe stricter or slightly different weighting than the implemented formulas.

Examples:

- MTF requirements are preset-adaptive, not always all-three alignment.
- Confidence implementation uses the formula documented in `SIGNAL_ENGINE.md`.

## Single Latest Zone State

The OB and FVG engines store the most recent bullish and bearish zones rather than maintaining arrays of many historical zones.

Impact:

- The current system is simpler and lighter.
- It does not provide a multi-zone institutional map.

## TradingView Dependency

The strategy depends on TradingView behavior for:

- Pine Script v6 compilation.
- `request.security` data availability.
- symbol availability for BTC, dominance, market cap, and OI references.
- alert delivery.
- chart object rendering limits.
