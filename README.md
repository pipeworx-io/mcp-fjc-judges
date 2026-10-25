# @pipeworx/fjc-judges

Biographies and appointment records of every US federal (Article III) judge since
1789 — Supreme Court justices, court of appeals and district court judges — from
the Federal Judicial Center's Biographical Directory.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1879+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `fjc_judge_lookup(name)` — one judge's full record: every appointment with court,
  title, appointing president and party, ABA rating, nomination / hearing /
  confirmation / commission dates, Senate vote, chief-judge and senior-status
  dates and how service ended; plus birth, death, education and career.
  Answers "who appointed Judge X", "when was X confirmed", "is X still serving".
- `fjc_judges_by_court(court, status?)` — judges of one court (e.g. "Ninth
  Circuit", "Southern District of New York", "Supreme Court"). `status` is
  `active` (default — active service only), `serving` (active or senior) or `all`.
- `fjc_judges_by_president(president, court_type?)` — every Article III
  appointment a president made, with totals by court level. Joseph R. Biden:
  235 appointments of 230 distinct judges (one judge can be appointed twice,
  e.g. Ketanji Brown Jackson to the D.C. Circuit and then the Supreme Court).
- `fjc_seat_succession(name | seat_id, court?)` — the chain of holders of one
  judgeship in order, with the named predecessor and successor of the judge asked
  about ("who did Judge X replace"). Built from the directory's Seat ID, whose
  last two digits are the holder's sequence on that seat.

Every response carries `source`, `licence` and `data_as_of` (when the FJC last
regenerated the directory).

## Auth

Keyless.

## Data sources

- Federal Judicial Center, Biographical Directory of Article III Federal Judges,
  1789–present. Public domain (a work of the US federal government).

The FJC publishes the directory as a single export (~5 MB, ~4,000 judges,
regenerated daily), not as a query API. The pack reads it per isolate and keeps
the parsed rows for up to six hours, so the first call on a cold isolate takes
about a second and the rest take milliseconds.

Things the next person would otherwise rediscover:

- One row per judge, with up to six appointments flattened into numbered
  columns (`Court Name (1)` … `(6)`). A judge elevated from district court to a
  circuit has two appointments; counts "by president" are appointments, and the
  response reports distinct judges alongside.
- Empty cells are frequently a single space, not empty — trim before testing.
- "Active" means no termination date and no senior-status date on that
  appointment. Senior judges still hear cases; use `status: "serving"` to
  include them.
- Presidents are written as the FJC writes them ("George H.W. Bush", "Joseph
  R. Biden"). A surname that matches two presidents returns
  `ambiguous_president` with both candidates rather than merging them.
- Magistrate, bankruptcy and state judges are not Article III judges and are
  not in this directory.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "fjc-judges": {
      "url": "https://gateway.pipeworx.io/fjc-judges/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/fjc-judges/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1879+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/fjc_judge_lookup \
  -H 'Content-Type: application/json' \
  -d '{"name":"Ketanji Brown Jackson"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/fjc_judge_lookup`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "fjc-judges": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-fjc-judges"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-fjc-judges
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Fjc Judges data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
