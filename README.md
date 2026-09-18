# 4dayweek.io MCP server

A free, read-only [Model Context Protocol](https://modelcontextprotocol.io) server over
[4dayweek.io](https://4dayweek.io), a job board for roles with genuinely reduced working hours:
4-day weeks, 9-day fortnights, half-day Fridays and more.

Connect it to Claude, ChatGPT, Cursor, Codex or any other MCP client and your assistant can search
live listings and employers directly, instead of scraping the site. There is no API key and no
sign-up.

| | |
|---|---|
| Endpoint | `https://4dayweek.io/api/mcp` |
| Transport | Streamable HTTP (stateless) |
| Authentication | None — anonymous, read-only |
| Server name | `io.4dayweek/jobs` |
| Version | 1.3.0 |
| Documentation | https://4dayweek.io/mcp |

This repository holds the manifest and the documentation. The server itself runs on 4dayweek.io;
there is nothing to install or host.

## Add it to your assistant

**Claude Code**

```bash
claude mcp add --transport http 4dayweek https://4dayweek.io/api/mcp
```

**Claude (web and desktop)** — Settings → Connectors → Add custom connector, then paste
`https://4dayweek.io/api/mcp`. No authentication needed.

**ChatGPT** — Settings → Connectors → Advanced → Developer mode, then add a connector pointing at
`https://4dayweek.io/api/mcp`.

**Codex**

```bash
codex mcp add 4dayweek --url https://4dayweek.io/api/mcp
```

or in `~/.codex/config.toml`:

```toml
[mcp_servers.4dayweek]
url = "https://4dayweek.io/api/mcp"
```

**Cursor** — `.cursor/mcp.json` in your project, or `~/.cursor/mcp.json` globally:

```json
{
  "mcpServers": {
    "4dayweek": {
      "url": "https://4dayweek.io/api/mcp"
    }
  }
}
```

**VS Code** — `.vscode/mcp.json`:

```json
{
  "servers": {
    "4dayweek": {
      "type": "http",
      "url": "https://4dayweek.io/api/mcp"
    }
  }
}
```

**Anything else** — the transport is streamable HTTP, so any MCP client can reach it directly:

```bash
curl -X POST https://4dayweek.io/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{
        "protocolVersion":"2025-06-18","capabilities":{},
        "clientInfo":{"name":"curl","version":"1.0"}}}'
```

## As a Claude Code plugin

This repository doubles as a Claude Code plugin marketplace, so the connector can be installed by
name and listed alongside your other plugins:

```
/plugin marketplace add 4dayweek/mcp
/plugin install 4dayweek@4dayweek-io
```

The `claude mcp add` line above does the same job in one command; the plugin route just keeps it in
your plugin list and updates with this repository.

## Tools

Six tools. Every one is read-only and says so in the protocol, so a cautious client can call them
freely.

| Tool | What it returns |
|---|---|
| `search_jobs` | Live listings, filtered by schedule type, category, country, work arrangement, seniority, skills, work-life score or posting date. |
| `get_job` | One listing in full by slug, including hours, locations and a published salary. |
| `search_companies` | Employers by working-hours policy, country, office-presence policy and whether they hire worldwide. |
| `get_company` | One employer's full profile: schedule policy, weekly hours, office policy, benefits and work-life score — plus day off, vacation and awards where the employer has published them. |
| `get_market_stats` | Aggregate market data across tracked employers, carrying an as-of date and its own caveats. |
| `explain_schedule_type` | Precise definitions: whether a schedule genuinely reduces weekly hours, and whether that is at full pay. |

## Try it

```
Find me remote 4-day-week engineering jobs open to applicants in the UK.
What is a 9-day fortnight, and is it the same as a compressed week?
Which employers run a genuine 4-day week at full pay, and how many are there?
Tell me about AWIN's 4-day week, and how it was verified.
Show me senior product roles at 32 hours a week, posted in the last fortnight.
```

## A compressed 4x10 is not a 4-day week

This is the distinction the tool descriptions exist to carry. A compressed week packs the same 40
hours into four days; a 4-day week is 36 hours or fewer with no cut in pay. Every listing carries an
explicit schedule type, and `explain_schedule_type` states, for any schedule, whether it genuinely
reduces weekly hours and whether that reduction is at full pay — so an assistant reading these tools
gets it right without being prompted.

## Limits and behaviour

- **Rate limit:** 300 requests per minute sustained, bursting to 60. `X-RateLimit-Limit` reports the
  burst ceiling.
- **Paging:** 10 results per page by default, 25 maximum, up to page 200. Responses carry `has_more`
  and `next_page`, and say so when the page ceiling is reached.
- **Read-only by construction:** the server can only reach read methods — a tool that writes would
  not compile. Nothing you send is stored as user data, and there is no account to create.
- **Applications happen on the site.** `get_job` returns the job's page on 4dayweek.io and whether
  the employer takes applications on-site or elsewhere; it never returns the employer's raw
  application URL.

## Privacy

The endpoint is anonymous. Each call is logged with the requesting IP, user agent, path, timestamp
and the name of the tool called, for abuse prevention and to see which tools are used — at most 14
days, in practice about two. No account, no cookies, no personal data.
Full policy: https://4dayweek.io/privacy

## Licence

The contents of this repository — the manifest, this documentation and the configuration snippets —
are MIT licensed (see [LICENSE](LICENSE) and [NOTICE](NOTICE)).

**That is not a licence to the data.** Results returned by the API and the MCP endpoint are covered
by [section 10 of our Terms](https://4dayweek.io/terms): a limited, revocable licence to use and
display them in your own product, conditioned on crediting 4dayweek.io and linking back, staying
within the rate limits, not misstating an employer's working-hours policy, and not using them to
build a competing job board. Our aggregate market statistics are separately published under
CC BY 4.0 on [our data page](https://4dayweek.io/data).

## Support

- Documentation: https://4dayweek.io/mcp
- Contact: https://4dayweek.io/contact
