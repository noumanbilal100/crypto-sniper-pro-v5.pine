# Crypto Sniper Pro v5

Professional repository for `crypto-sniper-pro-v5.pine`, a TradingView Pine Script v6 strategy.

The Pine strategy is preserved as the source of truth. Repository work in this pass only reorganized files and added documentation; entries, exits, signals, calculations, alerts, and trading logic were not changed.

## Repository Structure

```text
/
|-- crypto-sniper-pro-v5.pine
|-- README.md
|-- LICENSE
|-- .gitignore
|-- CONTRIBUTING.md
|-- SECURITY.md
|-- VERSION.md
|-- RELEASE_NOTES.md
|-- docs/
|   |-- ARCHITECTURE.md
|   |-- FEATURES.md
|   |-- MODULES.md
|   |-- SIGNAL_ENGINE.md
|   |-- SMART_MONEY.md
|   |-- ALERTS.md
|   |-- INPUTS.md
|   |-- ROADMAP.md
|   |-- CHANGELOG.md
|   |-- KNOWN_LIMITATIONS.md
|   `-- PERFORMANCE.md
|-- images/
|-- examples/
`-- tests/
```

## Source File

- File: `crypto-sniper-pro-v5.pine`
- Pine version: `//@version=6`
- Mode: `strategy(...)`
- Overlay: enabled
- Baseline SHA-256: `70603E70DF701ADC01CE8BBA05D78DD99D4A63410241A4B69103954E81D1B79C`
- Strategy defaults: `initial_capital=10000`, `commission_value=0.05`, `slippage=2`, `process_orders_on_close=true`, `calc_on_every_tick=false`

## What The Strategy Does

Crypto Sniper Pro v5 combines trend, smart-money structure, liquidity, momentum, volatility, sentiment, risk, execution, dashboard, and alert logic into one Pine strategy.

High-level flow:

1. Resolve preset thresholds and user inputs.
2. Build lower-timeframe and multi-timeframe trend state.
3. Detect external structure, sweeps, order blocks, fair value gaps, and HTF support/resistance.
4. Confirm setup quality with RSI, MACD, volume, CVD, BTC trend, crypto sentiment, funding, OI, session, volatility, and news filters.
5. Count confluences and compute confidence.
6. Validate reward:risk, cooldown, daily cap, date range, and daily drawdown.
7. Submit strategy entries and exits.
8. Render visual overlays, dashboard rows, signal banner, and alerts.

## Documentation

- `docs/ARCHITECTURE.md`: full architecture and data-flow report.
- `docs/MODULES.md`: module-by-module catalog.
- `docs/FEATURES.md`: feature matrix and operating status.
- `docs/SIGNAL_ENGINE.md`: long/short signal pipeline and gates.
- `docs/SMART_MONEY.md`: structure, liquidity, OB, FVG, HTF S/R, and Wyckoff behavior.
- `docs/ALERTS.md`: alert message formats and alertcondition map.
- `docs/INPUTS.md`: complete input group reference.
- `docs/PERFORMANCE.md`: risk, execution, metrics, and validation notes.
- `docs/KNOWN_LIMITATIONS.md`: observed current limitations.
- `docs/ROADMAP.md`: future engineering roadmap.
- `docs/CHANGELOG.md`: documentation and repository changelog.

## Presets

| Preset | Min confirmations | Min confidence | Cooldown | Min R:R | Max trades/day |
| --- | ---: | ---: | ---: | ---: | ---: |
| Strict (sniper) | 5 | 80% | 20 bars | 2.5 | 2 |
| Balanced | 3 | 65% | 10 bars | 2.0 | 3 |
| Aggressive | 2 | 50% | 5 bars | 1.5 | 6 |
| Scalping | 3 | 60% | 3 bars | 1.5 | 15 |
| Aggressive Scalping | 2 | 45% | 1 bar | 1.2 | 40 |
| Custom | User input | User input | User input | User input | User input |

## Usage

1. Open TradingView.
2. Open Pine Editor.
3. Paste or open `crypto-sniper-pro-v5.pine`.
4. Add the strategy to a chart.
5. Configure presets, filters, risk, alerts, and visuals from the script settings.

## Development Rules

- Preserve Pine Script v6 compatibility.
- Do not alter trading logic without an explicit strategy-change task.
- Keep documentation changes separate from logic changes.
- Validate any future source changes in TradingView Pine Editor.
- Treat alerts and webhook payloads as production interfaces.

## Disclaimer

This repository documents and packages a trading strategy. It is not financial advice. Backtest and live results are not guaranteed.
