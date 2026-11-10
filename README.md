# carddata

carddata is a small local bridge to the hosted
HobbyCardIndex card-prices MCP server, which tracks
current sold prices for sports and trading cards. You
can describe a card the way a collector would — year,
set, player or character, card number — and get back
matches with their raw and PSA 10 prices, then pull
one card in full to see what it sold for at every
grade: raw, PSA 7 through PSA 10, BGS 9.5, CGC 10 and
SGC 10. You can also read the plan catalogue to see
what the commercial API and app plans cost and cover.

The bridge holds no prices of its own. It answers the
tool list locally, exactly as captured from the live
endpoint, and forwards each tool call upstream,
returning whatever comes back untouched. The surface
here is three read-only tools. The original server
also offered a checkout tool; that tool is
intentionally not part of this rebuild, so nothing in
this package starts a purchase or handles payment.

## Tools

- `search_cards` — find a card and its current sold
  price. Required: `query` (year, set, player or
  character, and card number if known, 2 to 120
  characters). Optional: `category` (for example
  baseball, basketball, football, hockey, soccer,
  pokemon, magic, yugioh, one piece) and `limit`
  (1 to 5 matches, default 5). Returns up to 5
  matching cards with the raw (ungraded) and PSA 10
  price, a `card_id`, and the link to the card page.
  Needs a HobbyCardIndex API key.
- `get_card_prices` — current sold prices for one
  card at every grade. Required: `card_id` (the id
  returned by `search_cards`). Returns raw, PSA 7,
  PSA 8, PSA 9, PSA 10, BGS 9.5, CGC 10 and SGC 10
  prices in US dollars, with the link to the card
  page for recent sales and price history. Needs a
  HobbyCardIndex API key.
- `list_plans` — read the plans on offer right now.
  Takes no arguments and works without an API key.
  Returns the Commercial API sizes (requests, photo
  scans, sold-sale lookups, cert lookups) and the app
  plans Pro, Dealer and Enterprise, with monthly
  prices in US dollars, the terms and how to buy.
  This tool only reads the catalogue; it starts no
  purchase.

## Configuration

- `HOBBYCARDINDEX_MCP_URL` (optional) — override the
  upstream endpoint. Defaults to
  `https://hobbycardindex.com/api/v1/mcp`. (The
  override hook is this rebuild's own labelled
  reconstruction.)
- `HOBBYCARDINDEX_API_KEY` (optional) — when set,
  every upstream request carries an
  `X-API-Key: <key>` header, for the caller's own
  HobbyCardIndex API key. When it is not set, no key
  header is sent. (The header name is pinned by the
  original's registry record; the record pins no
  package env name, so forwarding the key from this
  env var is a labelled reconstruction — the original
  surface was captured keyless.)

### Forwarding mechanics

On the first tool call the client opens a streamable
HTTP session upstream: a JSON-RPC `initialize`
request (protocolVersion `2025-03-26`, clientInfo
name `carddata`), then a
`notifications/initialized` notification, then the
`tools/call` request carrying the caller's arguments
unchanged. When the upstream answers `initialize`
with an `Mcp-Session-Id` header, that value rides on
every later request of the session, which is reused
for the life of the process. Replies may arrive as
plain JSON or as server-sent events; both are read.

Successful upstream results pass through verbatim,
including any `isError` flag the upstream sets.
Failures — a JSON-RPC error, a non-2xx status, an
unreadable response, or a request that cannot be
completed — come back as an `isError` result whose
text starts with `carddata error:` (this rendering
is this rebuild's own labelled reconstruction).

## CLI

Each tool is also a subcommand taking one JSON
argument string:

```bash
npx @reneturcios/carddata search_cards '{"query":"1989 Upper Deck Ken Griffey Jr 1"}'
npx @reneturcios/carddata get_card_prices '{"card_id":12345}'
npx @reneturcios/carddata list_plans '{}'
npx @reneturcios/carddata --version
```

Exit codes: `0` on success, `1` on an `isError`
result, `2` on a usage error.

## MCP server

```bash
npx -y @reneturcios/carddata mcp
```

MCP client config:

```json
{
  "mcpServers": {
    "carddata": {
      "command": "npx",
      "args": ["-y", "@reneturcios/carddata", "mcp"]
    }
  }
}
```

## Docker

```bash
docker run -i ghcr.io/cloudcorprecords/carddata
docker run --rm ghcr.io/cloudcorprecords/carddata --version
```

The container's default command is `mcp`, so the first
form is a stdio MCP server; any other arguments reach
the CLI normally.

## Development

```bash
npm install
npm run build
npm test
npm run dev
```

The test suite runs entirely against a stubbed
JSON-RPC upstream — no live network calls.

## License

MIT — see `LICENSE`.
