# Sources, attribution, and licensing status

Reviewed **2026-09-07**. These Fluvion procedures were newly authored from the requested product facts, protocol documentation, and topic-level inspiration from **Orderly Network's skills project**. They are not a renamed wholesale copy, fork, or bundle of upstream source. No upstream code blocks, full procedures, logos, or license notice were copied.

## Upstream evidence inspected via web fetch

Repository: [OrderlyNetwork/skills](https://github.com/OrderlyNetwork/skills/tree/master/skills). At review, [master resolved](https://api.github.com/repos/OrderlyNetwork/skills/commits/master) to commit [`2440ff0b6f2e886b29fe9b27f22c78bbaf563ca4`](https://github.com/OrderlyNetwork/skills/commit/2440ff0b6f2e886b29fe9b27f22c78bbaf563ca4). The files were fetched from current master; this commit identifies the observed revision, not a guarantee that future master content is unchanged.

| Evidence | Observed result |
| --- | --- |
| [Current README](https://raw.githubusercontent.com/OrderlyNetwork/skills/master/README.md) | HTTP 200; describes skills and docs MCP; License section says MIT |
| [Current package.json](https://raw.githubusercontent.com/OrderlyNetwork/skills/master/package.json) | HTTP 200; package version 0.5.0; author Orderly Network; license field MIT |
| [Repository root listing](https://api.github.com/repos/OrderlyNetwork/skills/contents/) | HTTP 200; no LICENSE or other license-named root file shown |
| [Raw LICENSE](https://raw.githubusercontent.com/OrderlyNetwork/skills/master/LICENSE) | HTTP 404; no license text obtained |
| [GitHub license API](https://api.github.com/repos/OrderlyNetwork/skills/license) | HTTP 404; no recognized license text obtained |
| [Onboarding skill](https://raw.githubusercontent.com/OrderlyNetwork/skills/master/skills/orderly-onboarding/SKILL.md) | HTTP 200; topic/navigation inspiration and docs-tool distinction |
| [Authentication skill](https://raw.githubusercontent.com/OrderlyNetwork/skills/master/skills/orderly-api-authentication/SKILL.md) | HTTP 200; request-signing concepts; stale timestamp advice not adopted |
| [Trading orders skill](https://raw.githubusercontent.com/OrderlyNetwork/skills/master/skills/orderly-trading-orders/SKILL.md) | HTTP 200; order lifecycle topics; unsafe float/holding shortcuts not adopted |
| [Positions skill](https://raw.githubusercontent.com/OrderlyNetwork/skills/master/skills/orderly-positions-tpsl/SKILL.md) | HTTP 200; position/risk topics; stale leverage write path and simplistic risk formulas not adopted |

### Licensing decision

**MIT is declared upstream, but a complete upstream license/copyright notice was not obtained from the checked locations.** A missing LICENSE or GitHub recognition is not proof the work is unlicensed; neither is a metadata label a substitute for preserving the applicable notice. We therefore did not wholesale copy or redistribute upstream material. We wrote original concise procedures, attributed the inspiration here, and retained only necessary factual protocol names, paths, headers, and terms.

No upstream copyright holder/year was invented and no MIT license was fabricated for this project. The owner has not selected a license for these new documents; this directory therefore makes **no new open-source license grant**. Before public distribution, the owner should select terms for its original content. Any future copying/adaptation of upstream protected content requires obtaining the applicable license and notices, checking scope, and preserving obligations (including MIT copyright/permission notices if MIT applies). Attribution here does not replace those obligations. Protocol documentation was used for facts; no broad license for that website is assumed.

Orderly remains named where necessary to identify the real protocol, upstream source, headers, endpoint hosts, or documentation MCP. Fluvion is the user-facing project name. No endorsement, ownership transfer, or affiliation claim is made.

## Current protocol references

These public documentation pages were read; the API hosts themselves were **not called**.

- [API authentication](https://orderly.network/docs/build-on-omnichain/api-authentication.md): exact signed bytes, base58 API seed example, header names, independent scopes, and 300-second clock tolerance.
- [Documentation index](https://orderly.network/docs/llms.txt): discovery and documentation MCP capability descriptions.
- [Account information](https://orderly.network/docs/build-on-omnichain/restful-api/private/get-account-information.md): `GET /v1/client/info`.
- [Current holdings](https://orderly.network/docs/build-on-omnichain/restful-api/private/get-current-holding.md): token holdings shape, not available margin.
- [All positions](https://orderly.network/docs/build-on-omnichain/restful-api/private/get-all-positions-info.md): position rows and account free-collateral/risk fields.
- [Create order](https://orderly.network/docs/build-on-omnichain/restful-api/private/create-order.md): body fields, margin mode, IOC behavior, acceptance versus execution, client ID uniqueness limited to open orders.
- [Order by client ID](https://orderly.network/docs/build-on-omnichain/restful-api/private/get-order-by-client_order_id.md): reconciliation by `GET /v1/client/order/{client_order_id}`. The hyphenated `get-order-by-client-order-id.md` candidate returned 404; the underscore form is the verified page.
- [Read leverage](https://orderly.network/docs/build-on-omnichain/restful-api/private/get-leverage-setting.md): singular GET path, symbol and margin mode.
- [Update leverage](https://orderly.network/docs/build-on-omnichain/restful-api/private/update-leverage-setting.md): plural POST path, symbol-specific versus all-symbol behavior, optional response data, and margin mode.

### Deliberate corrections and safeguards

1. Current auth docs specify a server-time difference of at most **300 seconds**, rather than the upstream skill's 30-second advice. Generate fresh timestamps anyway.
2. Leverage writes use **`POST /v1/client/leverages`** with `symbol` and `leverage`; the guides also specify the approved margin mode. Omitted symbol changes all symbols. Success may have no `data`; read back instead of resending.
3. Base58 API secret may have an optional `ed25519:` prefix; it must decode to a **32-byte API seed**, not a wallet private key. No raw-secret prompts/logs or copied credential examples.
4. Decimal arithmetic, pre-send notional/margin caps, pending risk, actual free collateral, and nonzero position filtering replace permissive shortcuts.
5. Per-order mainnet authorization, durable intent records, ID reconciliation after ambiguous sends, and separate reduce-only close authorization are local safety policy—not claims that the protocol enforces these on the agent's behalf.
6. Product broker is `fluvion`; [fluvion.io](https://fluvion.io), [docs.fluvion.io](https://docs.fluvion.io), and [API key UI](https://fluvion.io/portfolio/api-key) are user-provided product configuration, not independently tested deployment guarantees. Broker is not an arbitrary order parameter.

## Test provenance and limits

Prior integration checks recorded by the delegating agent on **2026-09-07** cover real read/info/holding/positions/orders, leverage, and an IOC fill **only**. The delegating agent reports executing these checks. This is prior integration evidence, not independent verification or a new execution by the author of these skills. No account IDs, keys, signatures, live payloads, balances, or fill records are reproduced.

No live API calls, git commits, or execution-tool installs were made for this project. No other directory was modified. Local validation checks Markdown destinations and YAML frontmatter; it does not test authentication, orders, closing, cancellation, modifications, withdrawals, registration, TP/SL, recovery, or every margin mode.

## Local navigation

[Project index](README.md) · [Onboarding](skills/fluvion-onboarding/SKILL.md) · [Authentication](skills/fluvion-api-authentication/SKILL.md) · [Account data](skills/fluvion-account-data/SKILL.md) · [Trading orders](skills/fluvion-trading-orders/SKILL.md) · [Positions/risk](skills/fluvion-positions-risk/SKILL.md)
