---
name: fluvion-positions-risk
description: "Inspect Fluvion positions and collateral, manage explicitly approved symbol leverage, and plan separately authorized reduce-only closes."
---

# Fluvion positions and risk

Use for exposure/risk reads or a precisely authorized change. No execution tool is included. Broker `fluvion`; [website](https://fluvion.io), [docs](https://docs.fluvion.io), [key UI](https://fluvion.io/portfolio/api-key).

## Read first

Default to read-only/testnet `https://testnet-api.orderly.org`; mainnet is `https://api.orderly.org`. Use an authorized secure signer and [authentication](../fluvion-api-authentication/SKILL.md), never raw secrets in chat/logs. Broker is account context, not an invented order parameter.

1. Read `GET /v1/positions`, account information, holdings, and relevant pending orders using [account data](../fluvion-account-data/SKILL.md).
2. Filter **open positions** by decimal `position_qty != 0`. Positive is long; negative is short. Keep symbol and margin mode together. Zero-quantity rows may retain pending-order or bookkeeping fields, not an open position.
3. Separate token holdings from risk-engine `free_collateral`, total collateral, isolated margin, and pending-order requirements. Never infer free collateral from holdings or assume all account collateral backs every isolated position.
4. Use decimal arithmetic. Approximate mark notional is `abs(position_qty) × mark_price`; label it an estimate. Unrealized PnL, funding, fees and unsettled amounts are distinct. Do not present entry-price-only PnL or a simple leverage liquidation formula as authoritative risk.
5. Report current margin metrics and protocol-provided liquidation estimates with timestamps and caveats. Missing fields mean unknown, not safe. Account-wide cross-margin effects and moving markets invalidate simplistic per-position guarantees.

## Symbol-specific leverage

A leverage change is a state change, not a read and not permission to open an order. Require explicit network/account, symbol, margin mode, target leverage, and acceptable margin/risk impact. Never silently switch CROSS/ISOLATED or adjust all symbols.

1. Read `GET /v1/client/leverage?symbol={symbol}&margin_mode={mode}`. Check current symbol/account limits, open positions, pending orders, and collateral.
2. Validate the requested integer leverage against current allowed limits. Changing leverage affects margin requirements and may be rejected for existing exposure; it does not resize or close a position.
3. Use **`POST /v1/client/leverages`** (plural) with **both `symbol` and `leverage`**, and explicitly include the approved `margin_mode`. Refuse to submit without `symbol`: omission enables an all-symbol update. Do not use the stale singular POST shown in the upstream skill.
4. Inspect HTTP status and `success`. A successful response may omit `data`; do not classify that alone as failure or resend the change. Read back the same symbol/mode to confirm leverage. On ambiguous result, inspect state rather than blindly retry.
5. Honor current rate limits; the reviewed documentation lists five leverage updates per 60 seconds per user. Never use a leverage update to evade an order cap or free-collateral check.

Reference: [update leverage](https://orderly.network/docs/build-on-omnichain/restful-api/private/update-leverage-setting.md) and [read leverage](https://orderly.network/docs/build-on-omnichain/restful-api/private/get-leverage-setting.md).

## Separately authorized reduction

Do not auto-close or cancel-all as cleanup, risk remediation, or a demonstration. Explain the risk and request a decision. An opening authorization does not authorize its closing trade.

For an explicit close, obtain full per-order scope: account/network, symbol, opposite side, close quantity or fraction, maximum notional, margin mode/budget, leverage, order type, price/slippage bounds, and approval expiry. Then:

1. Refresh the exact position and any competing close orders; stop if already zero or the position no longer matches approval.
2. Use SELL for a long and BUY for a short. Decimal close quantity must not exceed current absolute position size after conservative tick rounding. Do not round up to satisfy minimum size without permission.
3. Require `reduce_only: true` on the separately approved `POST /v1/order`. Preserve margin mode and enforce pre-send caps via [trading orders](../fluvion-trading-orders/SKILL.md). Never remove reduce-only to bypass rejection.
4. Follow order-ID reconciliation. Submission is not a fill, and partial fills can leave exposure. Report remaining position; do not automatically retry or reverse it.

## Out of scope and evidence

TP/SL, trailing stops, margin transfers, withdrawals, and automatic liquidation prevention are not implemented or tested here. Do not invent algo payloads or attach protective orders implicitly; obtain separate authorization and verify the current protocol documentation before designing those flows. No stop guarantees a fill or prevents all loss.

Prior integration checks recorded by the delegating agent on **2026-09-07** cover real read/info/holding/positions/orders, leverage, and an IOC fill only. Close orders, TP/SL, all-symbol leverage, isolated-mode flows and other risk operations are not claimed tested. No live calls were made to author this guide.

[Onboarding](../fluvion-onboarding/SKILL.md) · [Index](../../README.md) · [Sources](../../SOURCES.md)
