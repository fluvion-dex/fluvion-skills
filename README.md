# Fluvion skills

Original, concise agent procedures for **Fluvion**, broker `fluvion`.

- Website: [fluvion.io](https://fluvion.io)
- Product documentation: [docs.fluvion.io](https://docs.fluvion.io)
- API key UI: [Portfolio → API key](https://fluvion.io/portfolio/api-key)
- Protocol REST hosts: mainnet `https://api.orderly.org`; testnet `https://testnet-api.orderly.org`.

These are **skill guides only**: no implemented MCP server, CLI, signer, execution adapter, or installed tools. Reading a guide does not authorize transactions. The protocol is Orderly; its required endpoint names and authentication headers are intentionally unchanged. No affiliation or endorsement is implied.

## Choose a skill

| Skill | Use it for |
| --- | --- |
| [fluvion-onboarding](skills/fluvion-onboarding/SKILL.md) | Safe setup, broker context, selecting the next guide |
| [fluvion-api-authentication](skills/fluvion-api-authentication/SKILL.md) | API seed handling, exact request signing, auth diagnostics |
| [fluvion-account-data](skills/fluvion-account-data/SKILL.md) | Read account configuration, holdings, positions, orders |
| [fluvion-trading-orders](skills/fluvion-trading-orders/SKILL.md) | Scoped order review, pre-send caps, submission reconciliation |
| [fluvion-positions-risk](skills/fluvion-positions-risk/SKILL.md) | Risk interpretation, symbol-specific leverage, authorized reductions |

Start with onboarding. Load the relevant `SKILL.md` through your agent's documented local-skill facility, or open the file directly as procedural context. Preserve this directory layout so relative links resolve. No published installation repository is claimed; no package installation is necessary to read these files.

## Non-negotiable defaults

Use reads and testnet by default. Every mainnet order needs explicit authorization for its account/network, market, side, order type, quantity/notional cap, margin mode and margin budget, leverage, and price/slippage bounds. Validate and enforce the cap **before sending**, using decimal arithmetic. Unknown collateral or stale state means stop, not guess. Holdings are not free collateral; zero-quantity rows are not open positions.

Never request raw secrets in chat, prompts, logs, command arguments, screenshots, or documents. An API seed is not a wallet key. Do not auto-close positions or cancel-all. Reducing a position requires a separately authorized `reduce_only` order. Never blindly retry an ambiguous order submission: reconcile by its persisted ID first. A successful submission is not proof of a fill.

## Documentation tools versus execution

Upstream's [documentation MCP](https://mcp.orderly.network), described in its [documentation index](https://orderly.network/docs/llms.txt), exposes documentation lookup, not an authorization to read private accounts or execute trades. Its protocol-specific tool names must remain unchanged when referencing it. An execution tool, secure signer, and user authorization are separate requirements; none is supplied here. Never send credentials to a documentation server.

## Evidence and validation

Reviewed **2026-09-07**. Prior integration checks recorded by the delegating agent on that date cover real read/info/holding/positions/orders, leverage changes, and an IOC fill. These historical observations were not rerun or independently verified while authoring this project. Other flows are not claimed tested, including registration, withdrawals, cancellation, modifications, close orders, and TP/SL. No live API calls were made for this work.

See [SOURCES.md](SOURCES.md) for attribution, current-documentation corrections, the incomplete upstream license evidence, and the intentionally conservative licensing decision. Local Markdown links and skill frontmatter are validation targets; they do not demonstrate live trading correctness.
