# Changelog

Notable changes to the **AlgoTradeOne Signal Engine** indicator
(`indicators/algotradeone-signal-engine.pine`). Each version maps to a git tag
and a TradingView publication. The in-script `VERSION` constant must match the
latest entry here.

## v1.0 — 2026-06-28

First versioned release.

- **5-leg confluence:** UT Bot · Supertrend · LinReg candles · anchored VWAP ·
  double-smoothed Heikin-Ashi Trend Cloud (with a colour-confirmation filter).
  A BUY needs every enabled leg bullish; a SELL needs every enabled leg bearish.
- **Entries:** run-based (one per confluence run, first permitted bar; queued,
  never lost), flat-only, with optional alternation and same-session guards.
- **Exit engine:** ratcheting ATR trailing stop, optional initial hard SL %,
  momentum-reversal exit, Supertrend-flip exit, and intraday square-off.
- **Quality filters (all optional, default off):** ADX regime, ATR expansion,
  structure break, HTF EMA alignment, min distance from VWAP, sideways block.
- **Non-repainting:** every decision on bar close; no lookahead; structure
  look-backs exclude the current bar.
- **Webhook alerts:** `LONG_ENTRY` / `SHORT_ENTRY` / `LONG_EXIT` / `SHORT_EXIT`
  (minimal JSON signal events; relay owns sizing/risk).
- **Visuals:** dark, badge-style status dashboard; Heikin-Ashi trend cloud;
  confluence-zone shading.
