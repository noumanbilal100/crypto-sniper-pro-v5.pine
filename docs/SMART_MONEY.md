# Smart Money Engine

This document covers the structure, liquidity, order block, fair value gap, HTF support/resistance, pattern, and Wyckoff components.

## Components

The smart money layer includes:

- External structure pivots.
- BOS and CHoCH.
- Liquidity sweeps.
- Order blocks.
- Fair value gaps.
- HTF support/resistance.
- Breakout/retest and sweep-reversal patterns.
- Wyckoff phase classification.

## External Structure

The strategy uses `ta.pivothigh` and `ta.pivotlow` with `i_pivotLen` to track major external highs and lows.

Bullish structure requires:

- latest external high above previous external high
- latest external low above previous external low

Bearish structure requires:

- latest external high below previous external high
- latest external low below previous external low

BOS and CHoCH are detected when price closes beyond the latest external pivot by at least `i_bosMinATR` times ATR.

## Liquidity Sweeps

A high sweep requires:

- price trades above the latest external high
- price closes back below that external high
- sweep is near the external high by ATR proximity
- optional prior trend movement condition passes
- bar is confirmed

A low sweep mirrors that logic below the latest external low.

Sweeps feed the pattern engine and Wyckoff manipulation phase.

## Order Blocks

Bullish OB:

- current candle is a bullish impulse greater than `i_obImpulse * ATR`
- previous candle was bearish
- previous candle body becomes the bullish OB

Bearish OB:

- current candle is a bearish impulse greater than `i_obImpulse * ATR`
- previous candle was bullish
- previous candle body becomes the bearish OB

Freshness and age:

- OBs are invalidated when price returns through the far side of the block.
- OBs expire after `i_obMaxBars`.

## Fair Value Gaps

Bullish FVG:

- current low is above the high from two bars earlier
- gap size is at least `i_fvgMinATR * ATR`

Bearish FVG:

- current high is below the low from two bars earlier
- gap size is at least `i_fvgMinATR * ATR`

FVGs remain active until price fills through the stored gap boundary.

## HTF Support And Resistance

The strategy requests pivot highs and lows from `i_htfsrTF`, stores the latest two resistance and support levels, and checks whether current price is within `i_htfsrProxATR * ATR`.

HTF S/R is used by:

- confluence counts
- confidence score
- final signal gate
- dashboard
- chart lines

## Pattern Engine

Bullish pattern sources:

- breakout retest above previous external high
- sweep-low reversal
- bullish OB interaction
- bullish FVG interaction

Bearish pattern sources:

- breakout retest below previous external low
- sweep-high reversal
- bearish OB interaction
- bearish FVG interaction

The resulting `patternReason` is used in labels and alerts.

## Wyckoff Phase Engine

The phase engine classifies:

- `ACCUMULATION`
- `MANIPULATION`
- `MARKUP`
- `DISTRIBUTION`
- `MARKDOWN`
- `NEUTRAL`

It uses range state, ATR contraction, prior trend context, sweeps, volume spike, and EMA trend.

## Current Implementation Notes

- The `i_useSMC` input exists, but current signal formulas do not use it.
- OB and FVG interactions can still produce `patternBull` and `patternBear`.
- `phaseFavorsLong` and `phaseFavorsShort` are calculated but not used in final signal gates.
- `i_htfsrLookback` is declared but not used by the HTF S/R calculation.
