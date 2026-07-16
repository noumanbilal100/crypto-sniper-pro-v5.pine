# Performance And Risk

This document summarizes strategy execution assumptions, risk controls, and performance analytics.

## Strategy Defaults

The source declares:

- initial capital: 10000
- commission: 0.05 percent
- slippage: 2 ticks
- pyramiding: 1
- process orders on close: true
- calculate on every tick: false
- calculate on order fills: false

## Position Sizing

Position size is risk-based:

```text
riskAmount = strategy.equity * risk percent * streak multiplier
quantity = riskAmount / stop distance
```

The stop distance is calculated separately for long and short setups.

## Stop Logic

Long stops prefer:

- bullish OB bottom with ATR buffer
- otherwise latest external low with ATR buffer
- otherwise ATR fallback

Short stops prefer:

- bearish OB top with ATR buffer
- otherwise latest external high with ATR buffer
- otherwise ATR fallback

The final stop is bounded against an ATR-based fallback to avoid overly wide structure stops.

## Targets

Multi-target mode:

- TP1 at `i_tp1R`
- TP2 at `i_tp2R`
- runner managed by trailing stop

Single-target mode:

- target uses nearby external structure when available
- otherwise uses `minRR`

## Active Risk Gates

The final signal gates include:

- reward:risk threshold
- confidence threshold
- confluence threshold
- cooldown
- daily trade cap
- date range
- daily drawdown halt
- long/short permission
- no active news blackout
- weekend filter when enabled

## Streak Sizing

Win/loss streak logic adjusts risk size:

- loss streak can reduce size
- win streak can increase size
- default thresholds come from inputs

## Partial TP And Breakeven

When enabled, the strategy submits a breakeven stop after price reaches +1R.

## Trailing Stop

Long trail:

```text
close - ATR * i_trailATR
```

Short trail:

```text
close + ATR * i_trailATR
```

The trail only moves in the favorable direction.

## Analytics

Displayed:

- win rate
- profit factor
- Sharpe approximation
- max drawdown

Calculated:

- Sortino approximation
- Calmar ratio
- recovery factor
- expectancy
- expected value
- Kelly fraction
- Hurst proxy
- Z-score
- Bollinger percent B

Several calculated analytics are not currently displayed or used as gates. See `KNOWN_LIMITATIONS.md`.

## Validation Checklist

Before any future logic release:

- Compile in TradingView Pine Editor.
- Test a BTC pair and a non-BTC alt pair.
- Test long and short alerts.
- Test plain text and JSON alert modes.
- Test all presets.
- Confirm dashboard rows render.
- Confirm object counts remain under TradingView limits.
- Compare source hash before and after changes.
