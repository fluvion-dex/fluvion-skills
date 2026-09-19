---
name: fluvion-api-authentication
description: "Handle Fluvion API seeds securely, construct exact Ed25519 request signatures, and diagnose authentication with read-only checks."
---

# Fluvion API authentication

Use for API key setup or signature failures. This guide supplies no signer or execution tool. Broker: `fluvion`. Website: [fluvion.io](https://fluvion.io); [docs](https://docs.fluvion.io); [key UI](https://fluvion.io/portfolio/api-key).

## Preconditions and secrets

- Default to testnet `https://testnet-api.orderly.org`. Use mainnet `https://api.orderly.org` only for an explicitly scoped task. Account and key registrations are environment-specific.
- The wallet authorizes account/key management; routine REST signatures use a separate API key. Solana wallet signing and API signing both use Ed25519 but **must not share secret material**.
- Let the user enter the secret directly into a trusted local secret manager/signer. Never ask for raw secrets in chat or prompts; never log them, place them in command arguments, copy them into fixtures, or forward them to documentation tools. Do not inspect secret files merely to confirm configuration.
- Accept the API secret as base58 text with an optional leading `ed25519:`. Strip that prefix once, decode with a maintained base58 implementation, and require **exactly 32 bytes**. Reject malformed data, wallet keys, 64-byte keypairs, mnemonics, and hex; do not silently truncate or guess encodings.
- Inside the trusted signer, derive the public key and compare with the registered API public key. The public header form is `ed25519:` plus base58 public-key bytes. A public key is not a signing seed.
- Start with `read`; `trading` does not imply `read`. Confirm scope, expiry, and any IP restrictions without printing the credential bundle.

## Exact signing procedure

1. Allowlist the chosen HTTPS API origin. Do not follow a redirect with authentication headers to another host.
2. Finalize uppercase method, pathname, query string, and body before signing. Preserve actual query order and percent encoding. GET reads have no body.
3. For JSON writes, produce one immutable serialized body and send those **same bytes**. Never sign one serialization and let an HTTP library create another.
4. Generate a fresh Unix timestamp in **milliseconds** immediately before signing. Synchronize the clock. Current protocol docs reject differences greater than **300 seconds**, not the stale 30 seconds in the upstream skill. This is tolerance, not permission to reuse old requests.
5. Concatenate without separators: `timestamp + METHOD + pathname + queryIncludingQuestionMark + exactBodyOrEmptyString`. Do not include the origin, fragments, or an invented empty JSON body.
6. Sign the UTF-8 bytes with Ed25519 using the 32-byte API seed. Encode signature bytes as base64url (URL-safe, without padding).
7. Send the unchanged request with these exact protocol headers:

| Header | Value |
| --- | --- |
| `orderly-account-id` | The intended registered account ID |
| `orderly-key` | `ed25519:` followed by base58 API public key |
| `orderly-timestamp` | The exact signed millisecond timestamp as text |
| `orderly-signature` | Base64url Ed25519 signature |
| `Content-Type` | `application/x-www-form-urlencoded` for GET/DELETE; `application/json` for POST/PUT |

Never rebrand these headers. Broker `fluvion` belongs in supported account/application configuration, not an invented order field.

## Verify with a read, not a trade

With an existing authorized secure execution tool, begin with `GET /v1/client/info`. Inspect HTTP status and JSON `success`; do not treat every HTTP 200 as success. Return only a sanitized readiness result. Follow [account reads](../fluvion-account-data/SKILL.md) for holdings, positions, and orders.

For failures check clock/units, exact signed query/body, uppercase method, key/account/environment match, expiry, scope, and IP restrictions. Redact signatures and identifying account details from shared diagnostics. Never weaken TLS, key validation, or scope controls to make a request work.

Re-signing a read after correcting a diagnosed error can be appropriate within rate limits. Re-signing a timed-out order is **not** a harmless auth retry: an ambiguous write must be reconciled by ID under [trading orders](../fluvion-trading-orders/SKILL.md), never blindly resubmitted.

## Navigation and evidence

[Onboarding](../fluvion-onboarding/SKILL.md) · [Positions/risk](../fluvion-positions-risk/SKILL.md) · [Index](../../README.md) · [Sources](../../SOURCES.md)

Primary reference: [current API authentication documentation](https://orderly.network/docs/build-on-omnichain/api-authentication.md). Prior integration checks recorded by the delegating agent on 2026-09-07 cover reads/info/holding/positions/orders, leverage, and an IOC fill only; no new calls or key-management tests were performed for this guide.
