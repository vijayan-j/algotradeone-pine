# Pine Trading Lab — Claude Context

Project for developing, backtesting, and shipping TradingView **Pine Script v6**
indicators and strategies for NSE intraday/positional trading. Signals generated
here fire JSON webhook alerts consumed by the execution server
(AlgoTradeOne → Zerodha Kite).

---

## Cardinal rule — separation of concerns

**Pine Script owns signal generation ONLY. The server owns ALL execution logic.**

- Pine scripts compute signals and emit a JSON payload via `alert()`.
- Pine scripts **never** carry live position sizing, risk, broker, or
  order-routing logic. Sizing/risk may be *simulated* inside `strategies/` for
  backtest validation only — never inside `indicators/`.
- AlgoTradeOne is the sole write/execution path. **Nothing in this repo places live
  orders.** The webhook payload expresses *signal intent*; the server computes
  quantity, product, order type, and risk.

---

## Repository map

| Path            | Purpose |
|-----------------|---------|
| `indicators/`   | `indicator()` scripts. Live signal sources; fire webhook alerts. **No order calls, no sizing.** Canonical live script: `ato-signal-engine.pine` (v3.2). |
| `strategies/`   | `strategy()` scripts. Backtest/validation twins of indicators. Simulated sizing + daily-loss circuit breaker live here. |
| `libraries/`    | `library()` modules. Reusable signal primitives (UT Bot, Supertrend, LinReg candles, anchored VWAP). Imported via `import <user>/<lib>/<ver>`. |
| `alerts/`       | Canonical JSON payload templates the server expects. Source of truth for the wire contract. |
| `backtests/`    | `raw/` = TradingView CSV exports · `reports/` = generated HTML/XLSX · `analysis/` = scripts that turn logs into reports. |
| `templates/`    | Boilerplate for new scripts. **Always start here.** |
| `docs/`         | Architecture, conventions, signal logic, webhook contract. |

---

## Workflow — adding a new indicator/strategy

1. Copy the matching file from `templates/`.
2. Implement signal logic per `docs/confluence-logic.md`.
3. Follow `docs/pine-conventions.md` (naming, sections, input groups, no-repaint).
4. Emit alerts using the **exact** payload in `docs/webhook-contract.md` / `alerts/`.
5. For any signal change, update the `strategies/` twin and re-run the backtest
   **before** the indicator is considered shippable. Indicator and strategy
   signal logic must stay identical — divergence is a bug.

---

## Hard constraints (do not violate)

- `//@version=6` on every script.
- Alerts use `alert.freq_once_per_bar_close`; signals confirm on **bar close**
  (no intrabar repaint).
- No `request.security(..., lookahead=barmerge.lookahead_on)` on past-sensitive
  series; HTF series read one bar back (`[1]`); structure look-backs exclude the
  current bar.
- **Flat-only** position model: when the exit engine owns the position, entries
  fire only while flat — no silent stop-and-reverse. Entry is **run-based**
  (first permitted bar of each confluence run; queued, never lost).
- Webhook payload is a **minimal signal event** — `event` / `signal_id` /
  `transaction_type` / `symbol` / `price` / `interval` / `time` only. **No
  execution fields** (qty, order type, product, routing, risk) — the relay owns
  all of those. Schema is defined in `docs/webhook-contract.md`; any change must
  be mirrored in `f_payload()` **and** `alerts/*.json` in the same commit.
- `signal_id = symbol-interval-bartime` is the relay's idempotency key — don't
  change its composition without updating the relay.
- Chart markers and `alert()` fires use the **same gated variables** — keep them
  in lockstep.

---

## Conventions (summary — full detail in `docs/pine-conventions.md`)

- `camelCase` for variables/functions, `UPPER_SNAKE` for constants.
- Group inputs with `group=` and `inline=`; one `// ── Section ──` banner per
  logical block.
- Explicit types on `export` library functions.
- No magic numbers — promote to named inputs/constants.
- Strategy declarations set realistic `commission_*`, `slippage`, and
  `process_orders_on_close=true` so backtests reflect live fills.
