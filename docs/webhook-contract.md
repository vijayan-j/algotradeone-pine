# Webhook Contract (Pine → Relay)

Source of truth: `f_payload()` in `indicators/algotradeone-signal-engine.pine`. The
payload is a **minimal signal event** — it states what happened, on which symbol,
when, and at what reference price. Nothing else. The relay/server owns all
execution parameters.

## Payload schema

```json
{
  "event": "LONG_ENTRY",
  "signal_id": "RELIANCE-5-1719210600000",
  "transaction_type": "BUY",
  "symbol": "RELIANCE",
  "price": 2945.50,
  "interval": "5",
  "time": "2026-06-24 10:15"
}
```

## Fields

| Field | Pine source | Notes |
|-------|-------------|-------|
| `event` | literal per branch | `LONG_ENTRY` \| `SHORT_ENTRY` \| `LONG_EXIT` \| `SHORT_EXIT` \| `TREND_CHANGE`. |
| `signal_id` | `ticker + "-" + timeframe.period + "-" + str.tostring(time)` | Idempotency key. The relay dedupes on this — the same bar can re-deliver. |
| `transaction_type` | literal per branch | `BUY` \| `SELL` \| `NONE`. The order direction to place/close. |
| `symbol` | `syminfo.ticker` | Bare chart ticker; the relay maps it to the broker instrument. |
| `price` | `close` (`format.mintick`) | Reference close for logging / slippage reconciliation. Advisory. |
| `interval` | `timeframe.period` | Chart timeframe the signal fired on. |
| `time` | `str.format_time(time, "yyyy-MM-dd HH:mm", "Asia/Kolkata")` | Signal bar time, IST. |

## Event semantics

| Event | `transaction_type` | Actionable? | Relay behaviour |
|-------|--------------------|-------------|-----------------|
| `LONG_ENTRY`  | `BUY`  | ✅ | Open long. |
| `SHORT_ENTRY` | `SELL` | ✅ | Open short. |
| `LONG_EXIT`   | `SELL` | ✅ | Close long. |
| `SHORT_EXIT`  | `BUY`  | ✅ | Close short. |
| `TREND_CHANGE`| `NONE` | ❌ | Informational only — **no-op**. Logged, never traded. |

## Relay-owned (NEVER in the payload)

Quantity, order type (MARKET/LIMIT), product type (MIS/CNC/NRML), exchange
routing, and all risk. This is deliberate — the v3.2 changelog removed every
execution field from the payload. A bug in Pine can therefore emit a wrong
*signal* but can never carry a wrong *order parameter*.

## Invariants

- Chart markers and webhook fires use the **same gated variables** — what you see
  is what fires. Keep them in lockstep across edits.
- `signal_id` is `(symbol, interval, bar_time)` — the relay's dedupe contract.
  Don't change its composition without updating the relay.
- One TradingView alert, condition = **"Any alert() function call"**, pointed at
  the relay webhook.
- Any field change here must be mirrored in `f_payload()` and `alerts/*.json` in
  the same commit.
