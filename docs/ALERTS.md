# Alerts

Crypto Sniper Pro v5 supports plain text alerts, JSON webhook alerts, and TradingView alert conditions.

## Alert Input

`i_alertFormat` selects:

- `Plain Text`
- `JSON (webhook)`

## Runtime Alerts

The script calls `alert()` when final signals fire:

- `longSignal` sends `longMsg`
- `shortSignal` sends `shortMsg`

Frequency:

- `alert.freq_once_per_bar_close`

This matches the confirmed-bar strategy design.

## Plain Text Long Message

Fields included:

- signal direction
- ticker
- timeframe
- entry
- stop loss
- TP1
- TP2
- reward:risk
- confidence
- pattern reason
- confluence count

## Plain Text Short Message

The short message mirrors the long message with short-side stop, targets, R:R, confidence, pattern reason, and confluence count.

## JSON Long Message

The long JSON payload includes:

```json
{
  "action": "BUY",
  "symbol": "<ticker>",
  "tf": "<timeframe>",
  "entry": 0,
  "sl": 0,
  "tp1": 0,
  "tp2": 0,
  "rr": 0,
  "confidence": 0,
  "pattern": "<pattern>"
}
```

## JSON Short Message

The short JSON payload includes:

```json
{
  "action": "SELL",
  "symbol": "<ticker>",
  "tf": "<timeframe>",
  "entry": 0,
  "sl": 0,
  "tp1": 0,
  "tp2": 0,
  "rr": 0,
  "confidence": 0,
  "pattern": "<pattern>"
}
```

## Alert Conditions

The script exposes:

| Condition | Trigger |
| --- | --- |
| `Crypto Sniper LONG` | `longSignal` |
| `Crypto Sniper SHORT` | `shortSignal` |
| `Crypto Sniper ANY` | `longSignal or shortSignal` |
| `Daily DD Hit` | `dailyDD_hit` |

## Webhook Safety

- Do not include secrets in alert messages.
- Validate downstream receivers before connecting live execution.
- Keep the JSON schema stable for automation.
- Paper test alert bridges before live use.
