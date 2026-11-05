# Tariffs API MCP server: the tariff and HTS source for your AI agents

Give your AI agents [Tariffs API](https://tariffsapi.com): US import duty rates by HTS code and country of origin, including base (Column 1/2) rates and Chapter 99 additional measures such as Section 301, 232 and 338. Measures that have ended, such as the Section 122 surcharge, are returned only when `as_of` falls inside the period they were in force.

This repository holds configuration and docs only. The server is hosted by Tariffs API; there is nothing to install or run.

Full documentation: https://tariffsapi.com/api-documentation#mcp-integration

## Endpoint

| | |
|---|---|
| URL | `https://tariffsapi.com/api/v1/mcp` |
| Transport | Streamable HTTP |
| Auth | `Authorization: Bearer YOUR_API_KEY` (capital **B**; lowercase `bearer` is rejected) or `X-API-Key: YOUR_API_KEY` |
| Plan | MCP access needs the Developer plan or higher: $199/month, or $149/month billed yearly. See [pricing](https://tariffsapi.com/pricing) |

Get an API key at https://tariffsapi.com. MCP calls count toward your plan's monthly API calls. Never commit your key, and don't pass it in the URL.

## Tools

| Tool | What it does | Arguments |
|---|---|---|
| `ResolveTariffTool` | Full tariff resolution for one HTS code and origin | `hts`, `origin` (ISO-2); optional `as_of`, `include_unknown_effective_dates` |
| `BatchResolveTariffsTool` | The same for up to 200 HTS/origin pairs in one call | `requests` (≤200 `{hts, origin}`); optional `as_of`, `include_unknown_effective_dates` |
| `GetByHtsTool` | Product details and base duties for an HTS code, plus measures if an origin is given | `hts_number`; optional `origin`, `as_of` |
| `SearchProductsTool` | Keyword search over the published HTS schedule text | `query`; optional `limit` (≤100), `has_tariffs` |
| `ListAgreementsTool` | Trade agreements and their special-rate codes (USMCA, FTAs, GSP) | optional `code` |

All tools are read-only (annotated `readOnlyHint: true`). Tariffs API reports rates for HTS codes you already have; it does not classify products. `SearchProductsTool` matches schedule text and does not tell you which code to file.

## Example prompts

- "Using Tariffs API, what duties apply to HTS 8471.30.0100 imported from China today? Break down the base rate and each additional measure."
- "Search the HTS schedule for 'lithium-ion batteries', then resolve the top result for origin Vietnam."
- "Which trade agreements does Tariffs API know about?"

## Setup

Replace `YOUR_API_KEY` with your key in each example.

### Claude Code

```bash
claude mcp add --transport http tariffs-api https://tariffsapi.com/api/v1/mcp \
  --header "Authorization: Bearer YOUR_API_KEY"
```

### Claude Desktop

Edit `claude_desktop_config.json` (Settings → Developer → Edit Config), then restart Claude Desktop. This uses [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) to bridge to the remote server and needs Node.js:

```json
{
  "mcpServers": {
    "tariffs-api": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote",
        "https://tariffsapi.com/api/v1/mcp",
        "--header", "Authorization:${AUTH_HEADER}"
      ],
      "env": { "AUTH_HEADER": "Bearer YOUR_API_KEY" }
    }
  }
}
```

### Cursor

Add to `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project):

```json
{
  "mcpServers": {
    "tariffs-api": {
      "url": "https://tariffsapi.com/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

### Gemini CLI

Install as an extension (uses `gemini-extension.json` and `GEMINI.md` from this repo). The installer asks for your key and stores it in the system keychain. You can also skip that step and export `TARIFFSAPI_API_KEY` in your shell instead:

```bash
gemini extensions install https://github.com/Danrod16/tariffsapi-mcp
# later: gemini extensions config tariffsapi   (to change the key)
```

Or add it directly to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "tariffs-api": {
      "httpUrl": "https://tariffsapi.com/api/v1/mcp",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

### n8n

1. Add an **AI Agent** node and attach an **MCP Client Tool** node.
2. Endpoint: `https://tariffsapi.com/api/v1/mcp`
3. Server Transport: **HTTP Streamable**
4. Authentication: **Bearer Auth**, create a credential with your API key as the token.
5. Tools to Include: **All** (or pick the ones you need).

Full guide plus an importable workflow: [docs/n8n.md](docs/n8n.md), [examples/n8n-tariffsapi-agent.json](examples/n8n-tariffsapi-agent.json).

## Check your key

```bash
curl -s -X POST https://tariffsapi.com/api/v1/mcp \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Errors

| HTTP | Code | Meaning |
|---|---|---|
| 401 | `UNAUTHORIZED` | Key missing, invalid or inactive (or lowercase `bearer`) |
| 403 | `PLAN_UPGRADE_REQUIRED` | Your plan doesn't include MCP; upgrade to Developer or higher |
| 429 | `RATE_LIMITED` | More than 500 requests/min per key; wait for `Retry-After` |
| 429 | `MONTHLY_LIMIT_EXCEEDED` | You've used your plan's monthly calls |

## MCP Registry

`server.json` is the listing for the official [MCP Registry](https://registry.modelcontextprotocol.io).

## License

MIT, see [LICENSE](LICENSE).
