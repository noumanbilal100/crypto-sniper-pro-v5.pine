# Roadmap

This roadmap is documentation and planning only. It does not modify the current Pine strategy.

## Phase 1: Preserve Baseline

- Keep `crypto-sniper-pro-v5.pine` unchanged while documentation is reviewed.
- Use the current docs as the baseline architecture map.
- Confirm whether documented observed notes are intended behavior or pending implementation.
- Add a manual test checklist for a few symbols and timeframes.

## Phase 2: Clarify Existing Controls

Potential follow-up items:

- Decide whether `i_useSMC` should disable OB/FVG participation in `patternBull` and `patternBear`.
- Decide how `i_htfsrLookback` should affect HTF pivot storage or whether it should be removed from inputs.
- Decide whether `phaseFavorsLong` and `phaseFavorsShort` should influence confidence or remain visual context only.
- Decide whether the dashboard wording should reflect preset-adaptive MTF requirements.
- Decide whether confidence comments/tooltips should be aligned with the implemented formula.

## Phase 3: Wire Or Remove Dormant Analytics

The script calculates several values that are not currently used in final gates. Each should be intentionally wired, displayed, or removed in a future logic pass.

Candidates:

- `equityCurveOK`
- `maxDDshutdown`
- `antiRevengeOK`
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

## Phase 4: Refactor For Maintainability

If future code changes are approved, consider:

- Grouping repeated dashboard color/text logic.
- Extracting repeated long/short calculations into helper functions where Pine allows it.
- Separating calculation, gate, execution, and rendering sections more strictly.
- Naming all modules consistently around "engine", "gate", "state", and "visual".
- Adding a top-of-file changelog comment only after logic changes are made.

## Phase 5: Verification Plan

Suggested validation work before logic changes:

- Compile in TradingView Pine Editor.
- Confirm all inputs appear in the expected groups.
- Confirm alert JSON is syntactically valid for long and short examples.
- Confirm dashboard renders when `i_showDash=true`.
- Confirm objects stay under label, box, and line limits on busy charts.
- Confirm no unexpected trades occur when each master switch is disabled.
- Confirm preset thresholds match the documentation.

## Phase 6: Research Ideas

These are strategic ideas, not recommendations:

- Add explicit SMC enable/disable behavior.
- Add separate OB and FVG toggles.
- Track multiple live OB/FVG zones instead of only latest state.
- Add symbol-specific presets for BTC, ETH, large-cap alts, and low-liquidity alts.
- Add optional risk-off mode using sentiment and stablecoin dominance.
- Add dashboard rows for currently calculated analytics.
- Add alert fields for stop distance, quantity, preset, and blocker state.

## Non-Goals For This Documentation Pass

- No change to entries.
- No change to exits.
- No change to risk formulas.
- No change to alerts.
- No change to visuals.
- No change to Pine source formatting.
