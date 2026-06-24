# Confluence Logic — ATO Signal Engine v3.2

Reference spec for `indicators/ato-signal-engine.pine`. This documents what
the live indicator actually does — not an aspiration.

## Core rule

A BUY requires **every enabled leg bullish**; a SELL requires **every enabled leg
bearish**. All four legs are read on confirmed bars only (`barstate.isconfirmed`).

```
buyState  = utBull  and stBull  and lrBull  and vwapBull
sellState = utBear  and stBear  and lrBear  and vwapBear
```

`vwapBull/vwapBear` are forced `true` (neutral) when the VWAP leg is disabled, so
turning it off cleanly collapses the system to a 3-leg confluence.

## The four legs (with shipped defaults)

| Leg | Implementation | Defaults | Bull condition |
|-----|----------------|----------|----------------|
| **UT Bot** | ATR recursive trailing stop | key `2.0`, ATR `1` | `close > utStop` |
| **Supertrend** | `ta.supertrend(mult, atr)` | ATR `10`, mult `2.0` | `dir < 0` |
| **LinReg** | `ta.linreg(close, len)` vs SMA/EMA signal | reg `11`, sig `7`, SMA on | `lrClose > lrSignal` |
| **VWAP** | manual cumulative, anchor-reset | anchor `Session`, standard polarity | `close > vwap` |

Extracted faithfully into `libraries/VijiSignals.pine` for the strategy twin.

## Signal behaviour — run → at most one entry

- **Fire once per run** (default on): one signal at the *first permitted* bar of
  each confluence run. A setup born inside a blocked window is **queued**, not
  lost — it fires when the window opens.
- **Alternate sides** (default on): repeat same-side signals are muted until the
  opposite side fires. With the exit engine on, this latch **auto-re-arms** after
  each exit, so same-direction continuation re-entries are still allowed.
- **Reset alternation each session** / **Run must start same session**: intraday
  guards, both default off.

## Position model — flat-only, no auto-reversal

`pos ∈ {1 long, -1 short, 0 flat}`. When the exit engine owns the position
(`useExit`), a new entry can fire **only while flat** (`flatOK`). There is no
silent stop-and-reverse — a reversal is always exit-then-fresh-entry.

## Exit engine

| Exit | Default | Trigger |
|------|---------|---------|
| ATR trailing stop | **on** | ratcheting stop, `exitAtr 14 × 2.5`; never loosens |
| Initial hard SL % | off (0) | caps first-bar risk: initial stop = **tighter** of ATR trail and fixed % |
| Momentum reversal | off | full opposite confluence prints |
| Supertrend flip | off | Supertrend flips against the position |
| Intraday square-off | off | independent of `useExit`; flattens from cutoff → close (`session.islastbar_regular` backstop) |

Exits are **never** blocked by the session/quality filters — only entries are.

## Entry filters (entries only)

`longQuality / shortQuality = sessionOK AND adxOK AND atrXOK AND structOK AND
htfOK AND vwDistOK`. All quality filters default off; each independently demands
regime/structure confirmation (ADX floor, ATR expansion, prior-N-bar structure
break excluding the current bar, HTF EMA on the last *closed* HTF bar, min VWAP
distance).

## Non-repaint guarantees (do not regress)

- Every decision gated on `barstate.isconfirmed` (closed bar).
- `request.security(..., lookahead = barmerge.lookahead_off)` and HTF series read
  one bar back (`[1]`).
- Structure look-backs exclude the current bar (`ta.highest(...)[1]`).
- Alerts fire `alert.freq_once_per_bar_close`.

## Backtest parity

`strategies/ato-signal-engine.pine` (not yet built) must replicate this whole
model — not just the legs. The legs come from `VijiSignals.pine`; the entry latch,
flat-only gate, and exit engine must be ported so the historical test validates
the live signal rather than a re-implementation.
