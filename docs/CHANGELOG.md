# Changelog

## 2026-07-13 - Production Repository Structure

Created the production repository layout:

- Moved architecture documentation into `docs/`.
- Renamed source file to `crypto-sniper-pro-v5.pine` without changing source content.
- Added root repository files: `LICENSE`, `.gitignore`, `CONTRIBUTING.md`, `SECURITY.md`, `VERSION.md`, and `RELEASE_NOTES.md`.
- Added focused documentation for signals, smart money logic, alerts, inputs, known limitations, and performance.
- Added `images/`, `examples/`, and `tests/` directories.

No Pine trading logic was changed.

## 2026-07-13 - Documentation Baseline

Added documentation files:

- `README.md`
- `ARCHITECTURE.md`
- `MODULES.md`
- `FEATURES.md`
- `ROADMAP.md`
- `CHANGELOG.md`

Documented:

- Signal Engine
- Trend Engine
- EMA Engine
- Smart Money Engine
- Liquidity Engine
- Order Block Engine
- Fair Value Gap Engine
- Dashboard
- Alerts
- Risk Management
- Entry Logic
- Exit Logic
- Inputs
- Visual Components

No Pine trading logic was changed.

## Source Baseline: Crypto Sniper Pro v5.0

The documented source baseline is `crypto-sniper-pro-v5.pine`.

Baseline source characteristics:

- Pine strategy declaration with overlay enabled.
- Preset system: Strict, Balanced, Aggressive, Scalping, Aggressive Scalping, Custom.
- Adaptive MTF trend stack.
- External structure, BOS, CHoCH.
- Liquidity sweeps.
- Order block and fair value gap detection.
- HTF support/resistance confluence.
- RSI, MACD, volume, CVD confirmations.
- BTC correlation gate.
- Crypto sentiment engine.
- Funding and OI gates.
- Session, weekend, volatility, and news filters.
- Confluence count and confidence score.
- Risk-based position sizing.
- Multi-target exits, breakeven, trailing stop.
- Daily DD halt, cooldown, daily cap, streak sizing.
- Advanced analytics calculations.
- Chart visuals, signal banner, dashboard, and alerts.
