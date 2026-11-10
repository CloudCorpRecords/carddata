# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted HobbyCardIndex card-prices MCP server, serving the three built tools locally for tools/list and forwarding every tools/call upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when issued).
- Tools: `search_cards`, `get_card_prices` and `list_plans`. The original's checkout tool is intentionally not part of this rebuild.
- Upstream results pass through verbatim; upstream failures surface as `carddata error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). Upstream endpoint overridable via `HOBBYCARDINDEX_MCP_URL`; when `HOBBYCARDINDEX_API_KEY` is set it is sent as an `X-API-Key` header.
