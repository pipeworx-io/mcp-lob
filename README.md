# mcp-lob

Lob MCP — US address verification, bulk verification, and autocompletion

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `lob_verify_address` | Verify a single US address for deliverability and return USPS-standardized/corrected components (ZIP+4, DPV codes, county, lat/lon). Pass structured components (primary_line, city, state, zip_code) OR a single freeform `address` string — not both. Example: lob_verify_address({ primary_line: "185 Berry St", secondary_line: "Ste 6100", city: "San Francisco", state: "CA", zip_code: "94107", _apiKey: "your-key" }) or lob_verify_address({ address: "185 Berry St Ste 6100 San Francisco CA 94107", _apiKey: "your-key" }) |
| `lob_verify_bulk` | Verify a list of US addresses in one request. Each address is an object with primary_line, city, state, zip_code (secondary_line/recipient optional). Returns per-address deliverability + corrected components. Example: lob_verify_bulk({ addresses: [{ primary_line: "210 King St", city: "San Francisco", state: "CA", zip_code: "94107" }], _apiKey: "your-key" }) |
| `lob_autocomplete` | Autocomplete a partial US address. Given a primary-line prefix (and optional city/state/zip), returns up to 10 full address suggestions. Suggestions are candidates, not guaranteed deliverable — verify with lob_verify_address. Example: lob_autocomplete({ address_prefix: "185 B", city: "San Francisco", state: "CA", _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "lob": {
      "url": "https://gateway.pipeworx.io/lob/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/lob/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "lob": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-lob"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-lob
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Lob data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
