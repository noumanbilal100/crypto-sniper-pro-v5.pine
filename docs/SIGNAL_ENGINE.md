# Signal Engine

This document explains the live long and short signal pipeline in `crypto-sniper-pro-v5.pine`.

## Pipeline

```text
Preset thresholds
  -> trend pass
  -> MTF pass
  -> pattern pass
  -> candle pass
  -> external market filters
  -> confluence count
  -> confidence score
  -> risk/reward validation
  -> discipline filters
  -> longSignal / shortSignal
```

## Preset Thresholds

The preset resolves five operational thresholds:

| Variable | Meaning |
| --- | --- |
| `minConfirm` | Minimum enabled confluence confirmations. |
| `minConf` | Minimum weighted confidence score. |
| `cooldown` | Bars required between signals. |
| `minRR` | Minimum reward:risk. |
| `maxPerDay` | Maximum signals per day. |

## Core Long Gate

`longCore` requires:

- bullish trend pass
- bullish MTF pass
- bullish pattern
- bullish candle pass
- BTC long filter
- funding long filter
- OI long filter
- sentiment long filter
- weekend filter
- no news blackout
- confirmed bar

## Core Short Gate

`shortCore` requires:

- bearish trend pass
- bearish MTF pass
- bearish pattern
- bearish candle pass
- BTC short filter
- funding short filter
- OI short filter
- sentiment short filter
- weekend filter
- no news blackout
- confirmed bar

## Confluence Count

The strategy counts these enabled confirmations:

- RSI
- MACD
- volume
- structure
- session
- CVD
- HTF S/R
- volatility regime

Disabled modules are not counted as passing confirmations. They are excluded from both the pass count and active module count.

## Confidence Score

The implemented confidence score uses:

- MTF alignment: 25 points for full alignment, otherwise 8 points per aligned MTF.
- Pattern: 15 points.
- Candle confirmation: 10 points.
- Confluence count: 3 points per confirmation.
- HTF S/R proximity: 10 points.
- Sentiment: 5 or 10 points depending on directional support.

Scores are capped at 100.

## Final Long Signal

`longSignal` requires:

- long trading enabled
- `longCore`
- long confirmation count above threshold
- long reward:risk above threshold
- long confidence above threshold
- cooldown OK
- daily cap OK
- date range OK
- daily drawdown not hit
- HTF support confirmation

## Final Short Signal

`shortSignal` mirrors the long signal:

- short trading enabled
- `shortCore`
- short confirmation count above threshold
- short reward:risk above threshold
- short confidence above threshold
- cooldown OK
- daily cap OK
- date range OK
- daily drawdown not hit
- HTF resistance confirmation

## Signal Consumers

Final signals feed:

- `strategy.entry`
- `strategy.exit`
- signal labels
- bar coloring
- background flashes
- signal banner
- dashboard signal row
- runtime alerts
- alert conditions

## Important Notes

- Signals fire on confirmed bars.
- The strategy does not calculate on every tick.
- The dashboard blocker row reports the first missing long and short condition.
- HTF S/R is both a confluence item and a final gate when enabled.
