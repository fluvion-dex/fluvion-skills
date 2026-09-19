---
name: fluvion-trading-orders
description: "Review a single Fluvion order, enforce decimal risk caps before submission, and reconcile acceptance, fills, or ambiguous outcomes without blind retries."
---

# Fluvion trading orders

Use to plan or manage a specifically authorized order. This is a guide, not an implemented trading tool. Broker `fluvion`; [website](https://fluvion.io), [docs](https://docs.fluvion.io), [key UI](https://fluvion.io/portfolio/api-key).

## Authorization gate

Default to reads and testnet `https://testnet-api.orderly.org`. Mainnet `https://api.orderly.org` requires **explicit per-order authorization**, not a general request to test trading. Record:

- Account and network; exact market symbol and margin mode (`CROSS` or `ISOLATED`).
- BUY/SELL side, open/increase versus separately authorized reduction, and order type.
- Base quantity, maximum quote notional, margin budget, and allowed leverage.
- Price or slippage bound, approval expiry, and a single submission scope.

Ask for missing scope. Never select a live trade merely to demonstrate signing. Require an existing trusted signer with `read,trading`; do not request or log raw secrets. Apply [authentication](../fluvion-api-authentication/SKILL.md), including unchanged `orderly-*` headers. API seed is not a wallet key.

## Preflight before sending

1. Refresh [account data](../fluvion-account-data/SKILL.md), current orders, nonzero positions, symbol leverage and margin mode. Holdings are not free collateral. A zero-quantity row is not an open position, but pending-order exposure still matters.
2. Read current symbol rules using `GET /v1/public/info/{symbol}` and fresh market/book data through a documented read. Validate active market, quantity/price bounds, `base_tick`, `quote_tick`, minimum notional, dynamic price limits, and size/risk limits. Do not hardcode a listing or leverage ceiling.
3. Use decimal arithmetic or scaled integers—not binary floating-point modulo—for price, quantity, notional, fees, and margin checks. Preserve decimal precision through JSON serialization using the current endpoint's numeric field types.
4. Calculate conservative quote exposure from quantity and an authorized executable price bound. Compare with the explicit notional cap **before signing and sending**. Include existing/pending exposure where relevant. For SELL, a limit is a minimum execution price, not an upper notional bound; use a separately defensible upper valuation bound or stop if a hard notional cap cannot be guaranteed. Do not pretend a last price is a hard cap.
5. Validate margin budget and free collateral with fees/slippage headroom and current protocol risk rules. `notional / leverage` is only an estimate, not sufficient margin validation. Never raise leverage, use all holdings, or deposit funds to bypass a failed check.
6. Quantize conservatively without expanding approved size or price/slippage bounds. Recompute all caps after rounding. If minimum order size exceeds the cap, stop rather than increase it.
7. Persist a new unique `client_order_id` and the exact approved intent in a protected local execution journal before submission. Use at most 36 characters under current rules; do not reuse IDs across completed intents. Store no secrets or signed headers. Current protocol uniqueness among open orders is **not durable idempotency**.

## Submit one intent

Current creation endpoint: `POST /v1/order`. Core fields are `symbol`, `side`, `order_type`, and validated `order_quantity`; price-dependent types additionally require `order_price`. Include the approved `margin_mode`, unique `client_order_id`, and explicit `reduce_only` as appropriate. No runnable trade payload is supplied here.

Broker `fluvion` is account/application context, **not an arbitrary order body parameter**. Do not inject broker IDs into unsupported fields or repurpose `order_tag` to claim attribution; tags can affect referrals/fees.

IOC uses a limit price, can fill partially, and cancels unmatched quantity. LIMIT may rest; MARKET can fill partially under liquidity/protocol limits and does not promise a chosen price. FOK and POST_ONLY have distinct semantics. Never change order type to force a fill without renewed approval. Consult the [current create-order reference](https://orderly.network/docs/build-on-omnichain/restful-api/private/create-order.md) before using any untested type.

## Reconcile, never blindly retry

- Check both HTTP status and JSON `success`. On acceptance, record `order_id` and `client_order_id`; successful submission **does not mean filled**.
- For a timeout, lost connection, malformed response, or uncertain server failure after sending, mark the intent **UNKNOWN** and freeze resubmission. An HTTP error alone may not prove non-submission.
- Reconcile using `GET /v1/order/{order_id}` when known, or `GET /v1/client/order/{client_order_id}`. Compare account, symbol, side, quantity, type, timestamps, executed quantity, and status to the journal. Consult order/fill history and position changes as corroboration, not as substitutes for order identity.
- Use bounded read retries with current rate limits. A temporarily missing record is not proof of absence. Do not retry with the same or a fresh client ID until the original outcome is conclusively resolved and a new submission is explicitly authorized. Escalate unresolved ambiguity to the operator.
- Report requested versus executed quantity, average execution price when available, fees, and remaining/cancelled quantity. A cancelled IOC may still have a partial fill. Do not automatically replace an unfilled remainder.

## Cancellation and closing

No auto-close and no cancel-all, including after a test. A specifically requested cancellation may use documented `DELETE /v1/order?order_id={id}&symbol={symbol}` only after matching the exact authorized order. Cancellation can race fills; reconcile final state. Modification, cancellation, and close flows are not claimed tested.

A closing trade is a separate authorization with opposite side, refreshed size, and `reduce_only: true`; see [positions/risk](../fluvion-positions-risk/SKILL.md). Do not treat a successful opening order as permission to close it later.

[Onboarding](../fluvion-onboarding/SKILL.md) · [Index](../../README.md) · [Sources](../../SOURCES.md)

Evidence: prior integration checks recorded by the delegating agent on 2026-09-07 cover read/info/holding/positions/orders, leverage and an IOC fill only. No live API calls were performed here. Other order types, cancellations, modifications, closing, TP/SL, and recovery scenarios remain unverified by this project.
