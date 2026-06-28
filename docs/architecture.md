# Architecture

## Two planes

```
┌──────────────────────────────┐        ┌─────────────────────────────────┐
│   SIGNAL PLANE (this repo)    │        │   EXECUTION PLANE (server)      │
│                              │        │                                 │
│  Pine v6 indicator            │        │  AlgoTradeOne (sole write path) │
│   ├─ compute confluence       │  JSON  │   ├─ validate + authenticate    │
│   ├─ confirm on bar close     │ ─────▶ │   ├─ size position (risk eng.)  │
│   └─ alert() webhook          │ webhook│   ├─ map → broker order         │
│                              │        │   └─ Zerodha Kite placeorder    │
│  Pine v6 strategy (offline)   │        │                                 │
│   └─ backtest the same logic  │        │  State, retries, circuit break  │
└──────────────────────────────┘        └─────────────────────────────────┘
```

**Signal plane** is deterministic and stateless per bar: given OHLCV it emits a
signal. It knows nothing about account balance, open positions, broker, or qty.

**Execution plane** is the only component that mutates account state. It owns
sizing, product type (MIS/CNC/NRML), order type, SL/TP attachment, idempotency,
retries, and kill-switches. The signal payload is *intent*, not an order.

## Why the split

- **Testability** — signal logic is backtestable in isolation; execution logic
  is unit-testable without a charting platform.
- **Safety** — a bug in Pine can produce a bad *signal* but can never place a
  bad *order*; the server is the single audited choke point.
- **Portability** — the same JSON contract lets any signal source (Pine,
  Chartink relay, screener) drive the same execution path.

## Indicator vs strategy in this repo

| | `indicators/` | `strategies/` |
|---|---|---|
| Pine type | `indicator()` | `strategy()` |
| Purpose | live signals → alerts | offline validation / backtest |
| Sizing & risk | **never** | simulated only (fixed-fractional, circuit breaker) |
| Orders | none — emits `alert()` | `strategy.entry/exit/close` (simulated) |
| Relationship | source of truth for live signals | must replicate indicator signals exactly |

The strategy exists to *prove* the indicator's signal logic on historical data.
It is not the live path. Keep their signal blocks byte-for-byte identical — copy the
block verbatim so there is one source of truth (the canonical indicator is
self-contained, with no library dependency).
