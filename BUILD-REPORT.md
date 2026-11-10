# BUILD REPORT — carddata

Rebuild of `com.hobbycardindex/card-prices` v1.0.0 (remote-only)
as `@reneturcios/carddata` 0.1.0 / `io.github.CloudCorpRecords/carddata`.
**Labelled subset: 3 of the original's 4 captured tools built; 1 omitted.**

## Fixture conflicts

**None.** The brief, the spec prose, and the fixture agree on the three
built tool names and their required fields. Required fields as served,
taken from the fixture
(`harness/fixtures/hobbycardindex-tools-original.json`):

- `search_cards` — required: `query`
- `get_card_prices` — required: `card_id`
- `list_plans` — required: **none — the captured inputSchema carries NO
  `required` key**; served verbatim with no `required` key invented
  (asserted by a dedicated test: the served schema is exactly
  `{type: "object", properties: {}}`).

The fixture's tool literals (name, title, description, inputSchema —
including the `title` keys and every property constraint) are served
verbatim from `src/forward.ts`; the literals were generated
programmatically from the fixture JSON (parsed and re-serialised, not
retyped) and verified by the registration-integrity test (served
tools/list deep-equals the fixture's 3 tool objects) and again by the
parity harness.

## What was built

All 19 factory files under `builds/carddata/`:

- `package.json` (`@reneturcios/carddata` 0.1.0, bin
  `carddata` → `dist/cli.js`, deps exactly `@modelcontextprotocol/sdk ^1`
  + `zod ^3`; devDeps `@types/node ^20`, `tsx ^4`, `typescript ^5`)
- `server.json` (description exactly the prescribed 94-char string
  "Sports and trading card sold prices: search cards, read prices at every grade, and list plans.";
  npm entry carries `"version": "0.1.0"`; OCI entry
  `ghcr.io/cloudcorprecords/carddata:0.1.0` carries **no** version field)
- `Dockerfile` (node:20-alpine both stages; server-name label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/carddata"`
  matches `server.json`)
- `tsconfig.json` (fleet-identical, 318 bytes), `.gitignore`,
  `package-lock.json`, `README.md` (fresh prose), `LICENSE` (MIT,
  Copyright (c) 2026 Rene Turcios), `CHANGELOG.md` (single 0.1.0 entry,
  2026-10-28)
- `src/index.ts`, `src/cli.ts`, `src/mcp.ts`, `src/types.ts`,
  `src/forward.ts` (core; exports `VERSION = "0.1.0"`)
- `test/forward.test.ts`
- `.github/workflows/{ci.yml, publish.yml, publish-container.yml,
  registry-publish.yml}` (container workflow: IMAGE
  `ghcr.io/cloudcorprecords/carddata`, smoke assertion on server name
  `carddata`)

Plus `BUILD-REPORT.md` (this file) and the `parity-out/` harness output.

### Labelled omission

The original's 4th captured tool, `start_checkout` (a Stripe checkout
funnel that also returns a minted API key after payment), is **not in
the fixture and is not registered, implemented, or presented as
available anywhere in this build**. It appears only in the test suite,
as the subject of absence assertions: it is not among the served tool
names, `isKnownTool` rejects it, the MCP layer rejects it as an unknown
tool (MethodNotFound, before any network hop), and it is not a CLI
command (usage, exit 2). The README and CHANGELOG describe the surface
as the 3 built tools and note that the original's checkout tool is
intentionally not part of this rebuild. `list_plans` is kept because it
only reads the published plan catalogue — it takes no email, starts no
checkout, and charges nothing.

## Design

- Forwarding client to `https://hobbycardindex.com/api/v1/mcp`
  (pinned), overridable via `HOBBYCARDINDEX_MCP_URL` (labelled
  reconstruction). tools/call is forwarded as a streamable-HTTP
  JSON-RPC client: initialize (protocolVersion `2025-03-26`,
  clientInfo `carddata`) → notifications/initialized → tools/call,
  honouring `Mcp-Session-Id` when issued; session established lazily
  and reused per client. Plain-JSON and SSE replies both parsed; the
  matching-id message wins; upstream results pass through verbatim
  (including `isError` and extra fields such as `structuredContent`).
- When `HOBBYCARDINDEX_API_KEY` is set, an `X-API-Key` header is sent
  on every upstream request (header name pinned by the registry
  record's remote declaration; the env name and the forwarding are
  labelled reconstructions — the record pins no package env name and
  the capture was keyless). When unset, no key header is sent. No
  `Authorization` header is sent in either case. Both branches are
  covered by tests.
- Upstream failures (JSON-RPC error, non-2xx status, malformed body,
  throwing/timing-out fetch) surface as `carddata error:` text results
  (labelled reconstruction).
- Registration via the MCP SDK **low-level** `Server` API
  (`ListToolsRequestSchema` / `CallToolRequestSchema` handlers) serving
  the fixture literals exactly; no `McpServer` + zod registration
  (per the task brief, overriding TEMPLATE.md §2 item 12).
  `src/mcp.ts` also exports `createServer` / `startMcpServer` /
  `handleCallTool` so the MCP layer is testable without a transport;
  the CLI's `mcp` subcommand starts it over stdio.
- Core takes an injectable fetch (`ForwardDeps.fetchFn`), so all tests
  run against a stubbed JSON-RPC upstream. No live network calls in
  tests; no live call claimed anywhere. The built surface is read-only
  market data (card search, per-grade sold prices, plan catalogue);
  no checkout, payment, or key-minting tool exists in the surface —
  none was added.

## Lockfile (Route 3, clean from birth)

Generated in a temp dir (`/tmp/carddata-lock`) containing only the
new `package.json`, with no `node_modules` present:
`npm install --package-lock-only --ignore-scripts --offline`.
Verified: parses, `name == @reneturcios/carddata`,
**128 package entries**; placed into the build dir with matching
sha256 (`d9c663b6…dc9f8` on both sides: full value
`d9c663b604a8099fbaabf0e6e6ea878b230dd6d078d6e73554c0445a37ddc9f8`).
Then `npm install --offline --ignore-scripts` in the build dir for
`node_modules`.

## Verification (all in the build dir unless noted)

- `npx tsc --noEmit` → exit 0
- `npm run build` → real `dist/` (cli.js, forward.js, mcp.js, all
  non-empty, with `.d.ts` + source maps)
- `node dist/cli.js --version` → `0.1.0`; unknown subcommand → usage on
  stderr, exit 2
- `npm test` → **58 tests, 58 pass, 0 fail** (9 suites: registration
  integrity — served tools deep-equal the fixture's full 3 tool
  objects, names in order, exactly the fixture's keys incl. title,
  required fields per tool incl. the `list_plans` no-`required`
  assertion, every property key in order, search constraint literals,
  isKnownTool; labelled omission — the checkout tool absent from the
  served names, rejected by isKnownTool, rejected by the MCP layer as
  MethodNotFound with zero network hops, and not a CLI command;
  session establishment/reuse/session-id handling, clientInfo/protocol
  version; env URL override, explicit url override, explicit-over-env
  precedence, blank-env fallback; X-API-Key header present on every
  request when the key is set / absent when unset / no Authorization
  header in either case; verbatim argument forwarding for all three
  tools incl. a fully-specified search and an empty-arguments plans
  call; verbatim result pass-through incl. upstream isError and SSE;
  JSON-RPC/init/malformed/no-result/HTTP/throw/timeout failures →
  `carddata error:`; parseRpcBody JSON + SSE + matching-id + [DONE]
  cases; MCP-layer MethodNotFound / InvalidParams validation; CLI
  --version / -v / unknown exit 2 / bad-JSON exit 2 / `mcp extra`
  exit 2)
- Parity: `python3 harness/parity.py --original
  harness/fixtures/hobbycardindex-tools-original.json --rebuild
  "stdio:node builds/carddata/dist/cli.js mcp" --out
  builds/carddata/parity-out` (run from `~/workspace/mcp-rebuild-100`)
  → **PASS, parity 100.0, 3/3 identical, no missing/extra/schema
  diffs** (see `parity-out/parity.json`)
- Consistency sweep: package.json version = server.json top-level =
  npm-entry version = OCI tag = core `VERSION` = CHANGELOG heading
  (0.1.0); package.json `mcpName` = server.json `name` = Dockerfile
  label; server.json description measured at 94 chars; OCI entry
  confirmed to have no `version` field.

Clean-room: all prose and code written fresh for carddata; only the
mechanics were patterned on the sibling glassdata build, and only the
fixture's tool literals are served verbatim.
