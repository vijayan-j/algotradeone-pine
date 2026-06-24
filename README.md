# AlgoTradeOne Pine

TradingView **Pine Script v6** indicators & strategies for NSE intraday/positional
trading. Signal generation lives here; execution lives on the server
(AlgoTradeOne → Zerodha Kite).

## Layout

```
algotradeone-pine/
├── CLAUDE.md              # Project rules — auto-read by Claude Code / Cowork
├── README.md
├── .gitignore
├── docs/                  # Architecture, conventions, signal logic, wire contract
│   ├── architecture.md
│   ├── pine-conventions.md
│   ├── confluence-logic.md
│   └── webhook-contract.md
├── indicators/            # indicator() — live signal sources (fire alerts)
├── strategies/            # strategy() — backtest twins (sizing + risk sim only)
├── libraries/             # library() — reusable signal primitives
├── alerts/                # Canonical JSON payload templates (wire contract)
├── backtests/
│   ├── raw/               # TradingView CSV exports
│   ├── reports/           # Generated HTML / XLSX performance reports
│   └── analysis/          # Scripts: logs → reports
├── templates/             # Boilerplate — start here for every new script
└── .claude/commands/      # Custom Claude Code slash commands
```

## Conventions

One source of truth per concern: signals in Pine, the wire contract in
`alerts/` + `docs/webhook-contract.md`, execution on the server. Every Pine file
is v6, confirms on bar close, and never places orders. See `CLAUDE.md` for the
full ruleset.

## Using with Claude Code / Cowork

`CLAUDE.md` is loaded automatically as context. Slash commands in
`.claude/commands/` (e.g. `/new-indicator`, `/audit-pine`) scaffold and review
scripts against the conventions in `docs/`.
