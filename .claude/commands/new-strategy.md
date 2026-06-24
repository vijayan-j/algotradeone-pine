Scaffold the backtest strategy twin for indicator "$ARGUMENTS".

Steps:
1. Copy `templates/strategy.template.pine` to `strategies/<kebab-name>.pine`.
2. Copy the **exact** signal block from `indicators/<kebab-name>.pine` (or import
   the same `libraries/` primitives) so signals are identical — divergence is a bug.
3. Keep the simulated risk layer (fixed-fractional sizing + daily-loss circuit
   breaker) in this file only; it must not appear in the indicator.
4. Set realistic `commission_*`, `slippage`, and `process_orders_on_close=true`.
5. Note in a header comment which indicator version this validates.

Remind me to run the TradingView backtest and drop the CSV export into
`backtests/raw/` before treating the indicator as shippable.
