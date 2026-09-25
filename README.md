# @pipeworx/sfda-drugs

[Saudi Food and Drug Authority (SFDA)](https://www.sfda.gov.sa) MCP — Saudi Arabia's drug
regulator: registered human drugs with official SAR prices, current drug shortages, recent drug
approvals, clinical trials running in the Kingdom, and registered drug companies/manufacturers.
Keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `sfda_drug_search(query?, start_page?, pages_to_scan?)` — registered drugs with official prices
  (SAR). Scans a bounded window of the ~1,400-page list and filters locally by scientific or trade
  name; returns a `next_page` cursor to continue.
- `sfda_drug_details(nid, id)` — the full SFDA record for one list entry: register number, ATC
  code, package size, shelf life, price, and the rest of the details page.
- `sfda_drug_shortages(scientific_name?, trade_name?, registration_no?, agent_name?, page?)` —
  current shortage list; server-side filters work here.
- `sfda_drug_approvals(scientific_name?, trade_name?, page?)` — recent drug approvals, newest
  first.
- `sfda_clinical_trials(title?, drug?, sponsor?, protocol_number?, phase?, status?, site?, page?)`
  — SFDA-registered clinical trials with sponsor, phase, protocol number and hospital site.
- `sfda_drug_companies(name?, registration_no?, production_line?, country?, page?)` — registered
  drug companies and manufacturing sites worldwide.

## Auth

None. All five lists are public server-rendered HTML on `www.sfda.gov.sa` — no key, no login, no
CAPTCHA on the data pages. The formal API host (`developer.sfda.gov.sa`) was unreachable when this
pack was built and is not used.

## Notes and traps (verified live 2026-09-07)

- **The drugs-list search is broken on the SFDA side.** `/en/drugs-list` ignores every filter
  parameter (`ScientificName`, `TradeName`, …) on both GET and the site's own POST flow, in both
  locales — the parameter is echoed into the form and pager but never applied. `sfda_drug_search`
  therefore filters client-side over a bounded page window. The other four lists apply their
  filters server-side correctly.
- **`?page=0` is not the first page.** The bare URL is page 0; `?page=0` returns a different,
  unrelated slice. The pack omits the parameter for page 0.
- **Blank `approval_date` on the newest approvals is a source gap, not a scraping miss** — older
  pages (e.g. page 20) carry real dates like `2025-08-01`; the newest rows are published before
  their date is.
- **Arabic enum values in otherwise-English tables.** Shortage status (`غير متوفر` = not
  available), company status (`مسجلة` = registered), drug type (`بشرية` = human) and country names
  render in Arabic. The pack translates known enums to English and keeps the original in `*_ar`
  fields. The companies `country` filter only matches the Arabic value.
- **`Array` as a field value on details pages** is a rendering bug on the SFDA side (a PHP array
  printed raw); the pack returns those fields as `null`.
- **Certificate issue dates on the companies list are Hijri calendar** (e.g. `1430-03-01`), not
  Gregorian.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sfda-drugs": {
      "url": "https://gateway.pipeworx.io/sfda-drugs/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sfda-drugs/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/sfda_drug_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"zantac","pages_to_scan":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/sfda_drug_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "sfda-drugs": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sfda-drugs"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-sfda-drugs
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Sfda Drugs data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
