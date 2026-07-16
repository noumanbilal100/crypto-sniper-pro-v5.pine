# Engineering Backlog

Source baseline: `crypto-sniper-pro-v5.pine`  
Backlog basis: `docs/CODE_ANALYSIS.md` and current source review  
Baseline SHA-256: `70603E70DF701ADC01CE8BBA05D78DD99D4A63410241A4B69103954E81D1B79C`

This backlog is prioritized for Pine Script v6 compatibility, non-repainting behavior, signal quality, and production maintainability. It is a planning artifact only.

## Priority Model

| Priority | Meaning |
| --- | --- |
| Critical | Incorrect or misleading behavior that can materially affect trading, controls, or safety. |
| High | Strong quality, risk, performance, or maintainability improvement. |
| Medium | Useful improvement with limited urgency or optional behavior impact. |
| Low | Nice-to-have polish, documentation, or future-facing work. |

## 1. Critical Bugs

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Fix daily trade counter reset ordering | Critical | Restores intended `maxPerDay` behavior; prevents daily cap from carrying across days after first use. | Medium | Low | Yes | Yes |
| Wire `i_useSMC` into OB/FVG pattern eligibility | Critical | Makes Smart Money master switch truthful; reduces unintended OB/FVG-qualified trades when disabled. | Low | Low | Only when `i_useSMC=false`; default remains unchanged. | No for default settings; yes for disabled-SMC tests. |
| Decide and implement behavior for unused `i_htfsrLookback` | High | Prevents misleading HTF S/R configuration; makes input behavior match UI promise. | Medium | Medium | Yes if used to filter levels; no if deprecated/hidden. | Depends on chosen implementation. |
| Add guard for zero `rangeSize` in Wyckoff phase calculation | High | Prevents `na`/division instability on flat or synthetic markets. | Low | Low | Rare edge-case behavior only. | Usually no; yes on flat-data tests. |
| Validate TP percent totals for multi-target exits | High | Prevents invalid or unexpected exit sizing if TP1% + TP2% exceeds practical bounds. | Medium | Low | Only for problematic input combinations. | Yes for those configurations. |
| Clarify or persist entry-time risk distance for BE/pyramid logic | High | Avoids dynamic stop recalculation altering +1R thresholds after entry. | Medium | Medium | Yes | Yes |

## 2. Signal Quality Improvements

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Make SMC switch control OB/FVG pattern sources | Critical | Reduces false positives in non-SMC configurations; aligns behavior with user intent. | Low | Low | Only when disabled. | No by default. |
| Add optional OB/FVG freshness confirmation at signal time | High | Reduces late, stale zone entries and weak retests. | Medium | Medium | Yes when enabled. | Yes when enabled. |
| Add separate OB and FVG master switches | High | Lets users isolate which SMC source is improving or degrading performance. | Low | Low-Medium | Only if defaults differ or user disables one. | No if both default to current behavior. |
| Add optional structure alignment gate for pattern direction | High | Reduces counter-structure false positives in aggressive presets. | Medium | Medium | Yes when enabled. | Yes when enabled. |
| Add optional Wyckoff phase confidence integration | Medium | Uses already-computed phase context to improve quality scoring. | Medium | Medium | Yes if wired into confidence/gates. | Yes |
| Add sweep age limit as an input instead of fixed 5 bars | Medium | Improves tuning between scalping and swing modes. | Low | Low | Yes if default changes; no if default remains 5. | No if default remains 5. |
| Add HTF trend confirmation mode for sentiment/MTF drift control | Medium | Reduces signals based on developing HTF states. | Medium | Medium | Yes when enabled. | Yes when enabled. |
| Add symbol-class presets for BTC, ETH, majors, and low-liquidity alts | Medium | Improves defaults by market type and volatility profile. | Medium | Medium | Yes when selected. | Yes for new presets. |
| Add optional volume quality filter using relative volume plus candle spread | Medium | Filters low-quality volume spikes and thin-market moves. | Medium | Medium | Yes when enabled. | Yes when enabled. |
| Add optional confluence weighting by preset | Low | Better separates strict, balanced, aggressive, and scalping behavior. | Medium | Medium | Yes | Yes |

## 3. Risk Management Improvements

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Fix daily trade counter reset | Critical | Restores daily trade limit as intended. | Medium | Low | Yes | Yes |
| Wire `maxDDshutdown` into final signal gate behind an input | High | Adds portfolio-level safety beyond daily DD. | Medium | Low-Medium | Yes when enabled. | Yes when enabled. |
| Wire `antiRevengeOK` into final signal gate behind an input | High | Reduces revenge-trade clustering after losses. | Medium | Low | Yes when enabled. | Yes when enabled. |
| Wire `equityCurveOK` into sizing or entry gating behind an input | High | Reduces risk during equity drawdown regimes. | Medium | Medium | Yes when enabled. | Yes when enabled. |
| Persist entry-time SL, risk distance, TP1, TP2, and trail basis | High | Makes exits and BE logic deterministic after entry. | Medium | Medium | Yes | Yes |
| Add maximum position quantity or notional exposure cap | High | Prevents oversized trades on tiny stop distances. | Medium | Medium | Yes for capped trades. | Yes |
| Add minimum stop distance guard | High | Avoids invalid or unrealistic sizing when stop distance is too small. | Low-Medium | Low | Yes for edge cases. | Yes for edge cases. |
| Add pyramid exposure cap and TP/SL handling for `LONG+` / `SHORT+` entries | Medium | Makes hybrid accumulation safer and clearer. | Medium | Medium | Yes when accumulation enabled. | Yes when enabled. |
| Add max consecutive loss halt as optional discipline control | Medium | Limits drawdown clusters. | Medium | Low-Medium | Yes when enabled. | Yes when enabled. |
| Add session-specific risk multiplier | Low | Allows lower size in lower-quality sessions. | Medium | Medium | Yes when enabled. | Yes when enabled. |

## 4. Performance Optimizations

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Bundle BTC `request.security` calls into one tuple | High | Reduces security call overhead. | Low | Low | No if expressions match. | No |
| Bundle OI close/volume security calls | High | Reduces security call overhead. | Low | Low | No if expressions match. | No |
| Bundle sentiment source requests per symbol/timeframe | High | Major reduction in `request.security` overhead. | Medium | Medium | No if expressions match. | No |
| Remove unused security close requests for sentiment symbols | High | Reduces unnecessary external data calls. | Low | Low | No, because values are unused. | No |
| Store Bollinger standard deviation once | Medium | Removes duplicate `ta.stdev` calculation. | Low | Low | No | No |
| Store repeated dashboard performance stats once | Medium | Reduces duplicate strategy metric calculations. | Low | Low | No | No |
| Add optional advanced analytics toggle | Medium | Improves performance when analytics are not used. | Medium | Medium | No if default remains current. | No by default. |
| Reduce visual object deletion/recreation on last bar | Medium | Improves chart responsiveness. | Medium | Medium | No intended trading change. | No |
| Move heavy display-only strings behind visual toggles | Low | Reduces string work when visuals are disabled. | Low-Medium | Medium | No | No |

## 5. Code Refactoring

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Separate calculation, gate, execution, visuals, dashboard, and alerts into stricter sections | High | Improves maintainability and review safety. | Low-Medium | Medium | No intended change. | No |
| Introduce helper functions for repeated formatting and score logic | High | Reduces duplicated long/short code and tooltip construction. | Medium | Medium | No intended change. | No |
| Create named constants for colors, labels, thresholds, and row indexes | Medium | Reduces dashboard/visual maintenance risk. | Low | Medium | No | No |
| Normalize variable naming across engines | Medium | Improves readability and onboarding speed. | Low-Medium | Medium | No intended change. | No |
| Group all inputs near the top | Medium | Improves user-facing organization and code scanning. | Medium | Medium | No intended change, but Pine input ordering changes UI. | No trading impact. |
| Convert dormant analytics into a clearly isolated analytics section | Medium | Makes unused work explicit and easier to toggle. | Low | Low-Medium | No | No |
| Add internal comments for behavior-sensitive gates | Low | Reduces future accidental signal changes. | Low | Low | No | No |

## 6. User Experience

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Add separate OB/FVG visual and logic toggles | High | Gives users clearer control over zone display versus signal use. | Low-Medium | Medium | Only if logic toggles changed. | No if defaults preserve behavior. |
| Add preset summary text to dashboard | Medium | Helps users understand active strictness without reading inputs. | Low | Low | No | No |
| Add setup diagnostics for all major gates, not only first blocker | Medium | Reduces confusion when multiple gates fail. | Low | Medium | No | No |
| Add mobile-friendly compact dashboard mode | Medium | Better usability on small screens. | Low | Medium | No | No |
| Add advanced/novice input mode grouping | Medium | Reduces configuration overwhelm. | Medium | Medium | No if defaults unchanged. | No |
| Add chart labels for active news blackout reason and remaining window | Low | Improves situational awareness. | Low | Low-Medium | No | No |
| Add examples for recommended preset/timeframe combinations | Low | Better onboarding. | Low | Low | No | No |

## 7. Dashboard Improvements

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Correct dashboard/tooltips where they imply all-three MTF agreement | High | Prevents user misunderstanding of preset-adaptive MTF rules. | Low | Low | No | No |
| Add rows for dormant risk flags if they remain calculated | Medium | Makes hidden risk state visible. | Low | Low-Medium | No | No |
| Add compact/full dashboard mode | Medium | Improves readability and performance. | Low-Medium | Medium | No | No |
| Add confidence breakdown row | Medium | Shows how score was built and why a signal is close/far. | Low | Medium | No | No |
| Add current trade plan row for entry, SL, TP1, TP2, trail, and R:R | Medium | Improves trade monitoring. | Low | Medium | No | No |
| Add security/data availability status for BTC and sentiment symbols | Medium | Helps diagnose missing external data. | Low-Medium | Medium | No | No |
| Add active module count and disabled module warning | Low | Improves configuration transparency. | Low | Low | No | No |

## 8. Alert Improvements

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Add alert schema version to JSON payload | High | Stabilizes webhook integrations. | Low | Low | No trading change; alert payload changes. | No |
| Add unique signal ID using ticker, timeframe, bar time, and side | High | Enables idempotent automation and duplicate protection. | Low | Low-Medium | No trading change; alert payload changes. | No |
| Add preset, confidence components, confluence counts, and blocker state to JSON | Medium | Better downstream diagnostics and routing. | Low | Medium | No trading change; alert payload changes. | No |
| Add risk fields: quantity, risk amount, stop distance, TP percentages | Medium | Improves execution bridge compatibility. | Medium | Medium | No trading change; alert payload changes. | No |
| Add optional compact JSON mode | Medium | Lets webhook consumers choose smaller payloads. | Low | Medium | No | No |
| Add alert docs with exact JSON examples and version history | Medium | Reduces integration errors. | Low | Low | No | No |
| Add dedicated daily DD JSON alert payload | Low | Makes risk halt automation cleaner. | Low | Low | No trading change; alert payload changes. | No |

## 9. Documentation

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Add TradingView compile and smoke-test checklist | High | Creates repeatable validation workflow. | Low | Low | No | No |
| Add baseline backtest matrix template | High | Enables controlled comparison after changes. | Low | Medium | No | No |
| Add configuration guide by trader style | Medium | Helps users select presets and filters safely. | Low | Medium | No | No |
| Add alert/webhook integration guide | Medium | Reduces production automation mistakes. | Low | Medium | No | No |
| Add data dependency guide for BTC, dominance, TOTAL, TOTAL3, and OI symbols | Medium | Explains why signals may vary by account/data availability. | Low | Low-Medium | No | No |
| Add visual glossary for OB, FVG, CHOCH, BOS, sweeps, and dashboard rows | Low | Improves onboarding. | Low | Medium | No | No |
| Add release checklist requiring source hash and behavior classification | Low | Improves governance. | Low | Low | No | No |

## 10. Future TradePulse Integration

| Item | Priority | Expected impact | Risk level | Estimated effort | Changes trading behavior | Affects existing backtests |
| --- | --- | --- | --- | --- | --- | --- |
| Define TradePulse webhook payload schema v1 | High | Creates stable contract for automation. | Low | Medium | No trading change; alert payload changes when adopted. | No |
| Add signal idempotency key to alert payload | High | Prevents duplicate execution in TradePulse. | Low | Low-Medium | No trading change; alert payload changes. | No |
| Add TradePulse-specific risk fields | High | Lets downstream engine validate risk before order placement. | Medium | Medium | No trading change; alert payload changes. | No |
| Add paper-trading relay test plan | High | Reduces live deployment risk. | Low | Medium | No | No |
| Add TradePulse execution status feedback plan | Medium | Enables future reconciliation between TradingView signals and execution results. | Medium | High | No Pine behavior initially. | No |
| Add TradePulse profile mapping for presets and symbols | Medium | Allows downstream rules by pair, timeframe, and preset. | Medium | Medium | No if only metadata. | No |
| Add event logging export format | Medium | Enables post-trade analytics and model evaluation. | Low-Medium | Medium | No | No |
| Add portfolio-level risk handoff contract | Medium | Lets TradePulse enforce exposure, correlation, and account-level drawdown. | Medium | High | No Pine behavior initially. | No |
| Add future feedback fields for fills, slippage, and outcome tagging | Low | Prepares for closed-loop analytics. | Medium | High | No Pine behavior initially. | No |

## Suggested Implementation Order

1. Fix daily trade counter reset.
2. Wire `i_useSMC` into OB/FVG pattern eligibility while preserving default behavior.
3. Add a baseline backtest matrix and compile checklist.
4. Bundle security requests and remove unused external close requests.
5. Add schema version and signal ID to JSON alerts.
6. Persist entry-time risk plan for BE/pyramid/exit consistency.
7. Wire optional risk controls behind disabled-by-default inputs.
8. Improve dashboard accuracy and diagnostics.
9. Refactor repeated code after behavior tests exist.
10. Define TradePulse payload v1 and paper-trading relay workflow.

## Backtest Governance

Every implementation item should be labeled before work starts:

- No behavior change expected.
- Behavior change only when new input is enabled.
- Default behavior change.
- Alert payload change only.
- Visual/dashboard change only.

For behavior changes, compare before/after on at least:

- BTCUSDT 15m, 1h, 4h.
- ETHUSDT 15m, 1h, 4h.
- One high-liquidity alt 15m and 1h.
- One lower-liquidity alt 15m and 1h.
- Strict, Balanced, Aggressive, Scalping, and Aggressive Scalping presets.
