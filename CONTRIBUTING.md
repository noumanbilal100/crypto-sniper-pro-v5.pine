# Contributing

This project is currently structured as a production-ready Pine Script repository with a strict source-preservation policy.

## Ground Rules

- Do not change entries, exits, alerts, calculations, signals, or risk logic unless the task explicitly asks for a logic change.
- Preserve Pine Script v6 compatibility.
- Keep documentation-only changes separate from Pine source changes.
- Do not make financial performance claims without reproducible backtest context.
- Do not commit secrets, webhook URLs, exchange credentials, API keys, or private account data.

## Change Types

Documentation changes may update:

- `README.md`
- `docs/`
- `VERSION.md`
- `RELEASE_NOTES.md`
- repository governance files

Logic changes require a dedicated review path:

1. State the intended behavior change.
2. Identify affected modules.
3. Preserve a copy of the previous source hash.
4. Compile in TradingView Pine Editor.
5. Document changed inputs, alerts, and strategy behavior.

## Validation Checklist

Before proposing any Pine source change:

- Confirm the script compiles in TradingView.
- Confirm alerts still work in the selected format.
- Confirm dashboard renders without object-limit errors.
- Confirm long and short paths are both reviewed.
- Confirm the change is reflected in `docs/CHANGELOG.md` and `RELEASE_NOTES.md`.

## Documentation Style

- Prefer precise module names from `docs/MODULES.md`.
- Document actual behavior, not intended behavior.
- Call out known limitations instead of hiding them.
- Keep examples generic and avoid account-specific data.
