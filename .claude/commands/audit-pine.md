Audit the Pine script at the path in "$ARGUMENTS" against the project rules.

Check and report (with file:line references) any violation of:
- `//@version=6` present; correct declaration type for its folder
  (indicators → `indicator()`, strategies → `strategy()`, libraries → `library()`).
- Repaint risks: intrabar signals, missing `alert.freq_once_per_bar_close`,
  `lookahead_on` on price-sensitive series.
- Separation of concerns: NO sizing/risk/order logic in an `indicators/` file;
  sizing in `strategies/` is simulation only.
- Alert payload matches `docs/webhook-contract.md` field-for-field; no hard-coded
  `qty` in indicator payloads; no reliance on `{{placeholder}}` tokens inside
  `alert()`.
- Conventions in `docs/pine-conventions.md`: naming, section banners, grouped
  inputs, no magic numbers.
- For a strategy: realistic `commission_*`, `slippage`, `process_orders_on_close`.
- If a strategy twin exists, confirm its signal block matches the indicator's.

Output a severity-ranked list (Blocker / Warning / Nit) and propose concrete diffs.
