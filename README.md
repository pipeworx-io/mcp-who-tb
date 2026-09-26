# @pipeworx/who-tb

WHO Global Tuberculosis Programme data — burden estimates, case notifications
and programme financing for **217 countries, 2000 onward**.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1684+ live data sources.

## Tools

| Tool | Answers |
|---|---|
| `who_tb_estimates` | Estimated TB incidence, mortality, HIV co-infection, case detection rate |
| `who_tb_notifications` | Cases actually reported by national programmes |
| `who_tb_budget` | TB programme budgets and cost per patient treated |
| `who_tb_dictionary` | What `e_inc_100k`, `c_newinc`, `budget_cpp_mdr` actually mean |

## Estimates vs notifications

These answer different questions and are easy to confuse. **Estimates**
(`e_inc_*`) are WHO's modelled view of how much TB exists, with uncertainty
ranges. **Notifications** (`c_newinc`) are cases actually diagnosed and reported.
The gap between them is undiagnosed TB, and quoting one as the other misstates a
country's epidemic.

## The dictionary is a first-class tool

The collection publishes its own data dictionary defining every column in every
dataset. Field names like `e_inc_100k` and `budget_cpp_dstb` cannot be guessed,
and a wrong guess reports a wrong number silently — so it is exposed as a tool
rather than left in a footnote.

## Auth

None.

## Data sources

- WHO TME CSV endpoint — `https://extranet.who.int/tme/generateCSV.asp?ds=estimates`
  (also `notifications`, `budget`, `dictionary`)
- WHO Global Tuberculosis Report — <https://www.who.int/teams/global-tuberculosis-programme/data>

The endpoint takes only `ds=`; there is no server-side filtering, so whole files
are fetched and filtered in the pack, memoised per isolate.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "who-tb": {
      "url": "https://gateway.pipeworx.io/who-tb/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/who-tb/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1684+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/who_tb_estimates \
  -H 'Content-Type: application/json' \
  -d '{"country":"India","years":"2020-2023"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/who_tb_estimates`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "who-tb": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-who-tb"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-who-tb
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Who Tb data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
