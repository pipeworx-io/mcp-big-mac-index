# @pipeworx/big-mac-index

The Economist's Big Mac index: what a Big Mac costs in about 55 countries, in local currency and in dollars, and how over- or under-valued each currency is against the dollar, euro, pound, yen and yuan, raw and adjusted for GDP per person. Covers 2000 to the latest release, which comes out twice a year.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1851+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `big_mac_index(country?, date?, history?, limit?)`: one country's row for an index release (default the latest). The row has the local price, the dollar exchange rate, the dollar price, and the raw and GDP-adjusted valuation against USD, EUR, GBP, JPY and CNY. Set `history: true` to get every release since 2000. Omit `country` to list every country in one release. Answers "is the Swiss franc overvalued by the Big Mac index?"
- `big_mac_valuation(base_currency?, date?, adjusted?, limit?)`: every currency in a release ranked by its valuation against one base currency, most overvalued first. Answers "which currencies are cheapest by the Big Mac index?"

Valuations are percentages. `+45.4` means the local currency is 45.4% overvalued against the base, and `-30` means 30% undervalued. A date argument (`YYYY`, `YYYY-MM` or `YYYY-MM-DD`) resolves to the latest release on or before it. Every response carries `data_as_of` (the latest release date in the index), `source` and `licence`. A country that is not in the index returns `found: false` with the list of countries that are.

## Auth

Keyless.

## Data sources

- The Economist, Big Mac index (the big-mac-data repository, full index): prices, exchange rates and raw and GDP-adjusted valuations as The Economist computes them.

**Licence.** The big-mac-data repository is published under the **MIT License**, copyright 2018 The Economist Newspaper: "Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files … to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies". Every response carries the attribution. Each call reads the published index at request time.

Notes for the next person:

- Country names follow The Economist: "Britain", not "United Kingdom", and "Euro area" (ISO code `EUZ`) for the euro. Common aliases (UK, US, eurozone, UAE) are mapped.
- GDP-adjusted figures are published only for countries where The Economist has GDP data. Elsewhere they are `null`.
- Releases before 2012 are irregular (one or two a year, in January, April, May, June or July). From 2012 they come out in January and July. One July 2019 release is dated the 9th, and one January 2020 release the 14th.
- Venezuela's currency code changed over time (VEF to VES) in the source, so match it by name or `VEN`.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "big-mac-index": {
      "url": "https://gateway.pipeworx.io/big-mac-index/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/big-mac-index/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1851+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/big_mac_index \
  -H 'Content-Type: application/json' \
  -d '{"country":"Switzerland"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/big_mac_index`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "big-mac-index": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-big-mac-index"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-big-mac-index
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Big Mac Index data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
