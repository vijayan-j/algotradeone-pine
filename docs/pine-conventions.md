# Pine Script v6 Conventions

## File header
- First line: `//@version=6`.
- One declaration per file: `indicator()`, `strategy()`, or `library()`.
- Title prefix ties related files together, e.g. `AlgoTradeOne Signal Engine` /
  `AlgoTradeOne Signal Engine [S]` for the strategy twin.

## Naming
- `camelCase` — variables, functions, inputs (`atrLength`, `longSignal`).
- `UPPER_SNAKE` — script-level constants and fixed identifiers (session strings,
  hard thresholds). Most state in practice is lowercase series (`utStop`, `pos`).
- Booleans read as predicates: `isNewDay`, `tradingHalted`, `gatePassed`.
- No magic numbers — every threshold is a named `input.*` or constant.

## Structure
Use section banners and keep blocks in this order:

```pine
// ── Inputs ─────────────────────────────────────────────
// ── Calculations ───────────────────────────────────────
// ── Signal logic ───────────────────────────────────────
// ── Plots / visuals ────────────────────────────────────
// ── Alerts ─────────────────────────────────────────────
```

## Inputs
- Group with `group=` and lay related fields on one line with `inline=`.
- Provide `minval`/`maxval`/`step` and a tooltip for anything non-obvious.

```pine
keyValue = input.float(1.0, "Key Value", minval=0.1, step=0.1,
     group="UT Bot", inline="utbot",
     tooltip="Sensitivity multiplier on ATR trailing stop")
atrPeriod = input.int(10, "ATR", minval=1, group="UT Bot", inline="utbot")
```

## No-repaint discipline (non-negotiable)
- Confirm signals on **bar close**, not intrabar. Gate alerts with
  `alert.freq_once_per_bar_close`.
- For higher-timeframe pulls: `request.security(sym, htf, expr)` **without**
  `lookahead_on`, and reference the *confirmed* value (`[1]`/`barstate.isconfirmed`)
  when mixing timeframes.
- Never use future-leaking constructs (`lookahead_on` on price, `calc_on_order_fills`
  hacks) in the signal path.

## Libraries
- Annotate every export:

```pine
// @function Short description.
// @param src Source series.
// @returns [value, longSig, shortSig]
export myFn(series float src, simple int len) => ...
```

- Explicit qualified types on params (`series` / `simple` / `const`) so callers
  get clear contracts and the compiler can optimise.

## Strategy declarations (backtest realism)
Always model costs so the backtest is not a fantasy:

```pine
strategy(..., initial_capital=100000,
     commission_type=strategy.commission.percent, commission_value=0.03,
     slippage=2, process_orders_on_close=true, calc_on_every_tick=false)
```

`process_orders_on_close=true` aligns simulated fills with the bar-close signal
model used live.

## Alerts → wire contract
- Build the JSON in Pine from real values (`syminfo.ticker`, `close`,
  `str.format_time(...)`) — **not** TradingView `{{placeholder}}` tokens, which
  the `alert()` function does not substitute.
- Indicator payloads carry signal intent only; no `qty`. See
  `docs/webhook-contract.md`.
