---
name: fluvion-onboarding
description: "Set up Fluvion context safely and select guides for authentication, account reads, scoped orders, and position risk."
---

# Fluvion onboarding

Use when a user first connects an agent or asks where to start. This is procedural guidance, not an implemented CLI, MCP, or trading integration.

## Establish context

1. Identify the task: documentation, account reads, testnet rehearsal, or a specific authorized mainnet action. Default to reads/testnet; never infer mainnet permission from an existing key.
2. Confirm broker `fluvion`, the intended account, and environment. Mainnet REST is `https://api.orderly.org`; testnet REST is `https://testnet-api.orderly.org`. Do not silently switch environments or substitute another broker if setup fails.
3. Direct the user to [fluvion.io](https://fluvion.io), [product docs](https://docs.fluvion.io), and [API key management at /portfolio/api-key](https://fluvion.io/portfolio/api-key). The user performs wallet approvals in the trusted UI. Do not request their wallet seed or API secret.
4. Prefer a dedicated, expiring, read-only API key. Add `trading` only when required; `read` and `trading` are independent scopes. Asset access is outside these guides.
5. Have the user provision a local secret manager or secure signer out of band. Confirm only readiness, public-key match, environment, expiry, and scopes—not raw secret contents. The supported API secret representation is base58 with an optional `ed25519:` prefix, decoding to exactly a 32-byte API seed, never a wallet private key.
6. Follow [authentication](../fluvion-api-authentication/SKILL.md) and then [account data](../fluvion-account-data/SKILL.md). Stop at a sanitized read summary unless more was explicitly requested.

## Broker and tool boundaries

`fluvion` identifies the account's broker relationship in the supported registration/lookup and application configuration flows. It is **not an arbitrary order parameter**. Do not create a new broker, pay a registration fee, or inject broker/referral tags into orders as part of onboarding.

The upstream [Orderly documentation MCP](https://mcp.orderly.network) is optional documentation retrieval; it does not sign requests or execute these procedures. Do not rename required protocol tools or send it credentials. This project implements no execution tools and requires no installation command. Load these local files using the host agent's existing skill support.

## Before any state change

Obtain explicit per-order mainnet scope: account/network, exact market symbol, side, order type, quantity and maximum notional, margin mode/budget, leverage, price/slippage bound, and expiry of approval. Recheck caps before sending with decimal arithmetic. Holdings are not free collateral, and zero-quantity position rows are not open exposure. Missing scope means ask, not trade.

Never auto-close, cancel-all, alter leverage, change margin mode, or withdraw during setup. A `reduce_only` close requires separate authorization. If an order outcome is ambiguous, do not retry blindly; reconcile its recorded ID. Submission success is not a fill.

## Next steps

- [Trading orders](../fluvion-trading-orders/SKILL.md): one authorized order and lifecycle reconciliation.
- [Positions and risk](../fluvion-positions-risk/SKILL.md): risk reads and separately approved leverage/reductions.
- [Project index](../../README.md) and [source/licensing evidence](../../SOURCES.md).

Evidence boundary: prior integration checks recorded by the delegating agent on 2026-09-07 cover real read/info/holding/positions/orders, leverage, and an IOC fill only. Onboarding/account registration and other flows are not claimed tested. No live calls were made to author this skill.
