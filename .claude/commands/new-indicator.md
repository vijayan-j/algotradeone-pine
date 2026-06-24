Scaffold a new Pine v6 indicator named "$ARGUMENTS".

Steps:
1. Copy `templates/indicator.template.pine` to `indicators/<kebab-name>.pine`.
2. Set the `indicator()` title + shorttitle. There is no strategy/version
   constant — the webhook `signal_id`, `event`, and `transaction_type` fields are
   derived automatically by `f_payload()`.
3. Replace the placeholder EMA-cross signal block with the four-leg confluence
   model from `docs/confluence-logic.md`, importing primitives from
   `libraries/AlgoTradeOneSignals.pine` where possible.
4. Keep the alert `buildSignal()` payload exactly matching `docs/webhook-contract.md`.
5. Create the matching strategy twin in `strategies/<kebab-name>.pine` with the
   identical signal block (run `/new-strategy <name>` if it doesn't exist).

Enforce every rule in CLAUDE.md: v6, bar-close confirmation,
`alert.freq_once_per_bar_close`, no sizing/orders in the indicator.
