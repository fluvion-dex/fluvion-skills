---
name: fluvion-account-data
description: "Read Fluvion account information, token holdings, positions, and orders without confusing balances with available trading collateral."
---

# Fluvion account data

Use for account inspection and pre-trade snapshots. Read-only guidance; no API client is implemented. Broker `fluvion`; [website](https://fluvion.io), [docs](https://docs.fluvion.io), [API key UI](https://fluvion.io/portfolio/api-key).

## Start safely

Confirm intended account and environment. Default to reads/testnet `https://testnet-api.orderly.org`; mainnet is `https://api.orderly.org` and must be deliberately selected. Require `read` scope and [exact API authentication](../fluvion-api-authentication/SKILL.md). Never request or log secrets; a trusted signer handles the base58-encoded 32-byte API seed out of band. Do not pass private account data to a docs MCP.

## Snapshot procedure

1. Record environment and retrieval time. Read only data necessary for the task, from the same account and environment throughout.
2. Check HTTP status and `success` separately. Validate expected shapes. Missing data means unknown, not zero. Preserve numeric precision with decimal parsing; preserve IDs as lossless values rather than floating-point numbers.
3. Use these protocol reads:

| Read | Interpret as |
| --- | --- |
| `GET /v1/client/info` | Account configuration, fee rates, account-level leverage limits; not a guarantee of selected symbol leverage |
| `GET /v1/client/holding` | Token rows in `data.holding`, including holdings/frozen amounts; **not free collateral** |
| `GET /v1/positions` | Position rows in `data.rows` plus account risk totals such as `free_collateral` and `total_collateral_value` |
| `GET /v1/orders` | Order records; use current documented symbol/status/time/page filters for the question |

4. When listing active orders, a documented `status=INCOMPLETE` filter is useful; preserve it in the signed query. Follow documented pagination and state whether the listing is complete. An incomplete-order listing alone cannot reconcile a quickly filled IOC order.
5. Count an open position only where decimal `position_qty != 0`. Positive means long and negative means short. Zero-quantity rows may carry bookkeeping or pending-order fields; exclude them from open-position counts but retain relevant pending risk separately. Keep margin mode with symbol when interpreting exposure.
6. Report holdings, actual free collateral from account risk data, open positions, and pending orders as distinct categories. Do not label `holding - frozen`, wallet USDC, or `total_collateral_value` as spendable trading margin. Missing/stale `free_collateral` blocks order sizing.
7. Treat separately fetched responses as a non-atomic snapshot. Include freshness, partial pagination, missing fields, and observed discrepancies. Refresh before a write; an earlier balance is not an ongoing spend authorization.

## Output contract

Return a minimal report: selected environment, redacted account reference, retrieval time, token holdings, free collateral (or unknown), nonzero positions with side/size/margin mode, relevant order IDs/status/executed quantities, and warnings. Do not dump raw account responses, email fields, headers, or credentials.

A returned order ID proves identity, not execution. A `success: true` order response does not prove a fill; inspect executed quantity and final lifecycle state. Zero open orders does not mean zero positions or no historical fills.

## Boundaries and next steps

Do not place an order, change leverage, cancel-all, close positions, or move assets to fix an account report. Per-order mainnet authorization, margin/notional/leverage caps and decimal pre-send validation belong to [trading orders](../fluvion-trading-orders/SKILL.md). Separately authorized `reduce_only` closing and leverage belong to [positions/risk](../fluvion-positions-risk/SKILL.md).

[Onboarding](../fluvion-onboarding/SKILL.md) · [Index](../../README.md) · [Sources](../../SOURCES.md)

Prior integration checks recorded by the delegating agent on 2026-09-07 cover read/info/holding/positions/orders, leverage, and an IOC fill only. This guide was authored without live API calls; completeness across pagination, all filters, and all account modes is not claimed tested.
