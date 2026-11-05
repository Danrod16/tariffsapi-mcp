# Use TariffsAPI in n8n (MCP Client Tool)

Checked against n8n docs and source on 5 Nov 2026: node "MCP Client Tool" (`@n8n/n8n-nodes-langchain.mcpClientTool`, v1.2+). Fields: **Endpoint**, **Server Transport**, **Authentication**, **Tools to Include**.

## Prerequisites
- An n8n instance (Cloud or self-hosted) recent enough to have Server Transport (MCP Client Tool v1.2+)
- A TariffsAPI key on the Developer plan or higher ([pricing](https://tariffsapi.com/pricing))
- A chat model credential (e.g. OpenAI)

## Steps
1. **Create the credential.** Go to Credentials → New → **Bearer Auth**, paste your TariffsAPI key as the token, and name it `TariffsAPI`. (Alternatively, use **Header Auth** with name `X-API-Key` and your key as the value.)
2. **New workflow.** Add a **When chat message received** trigger.
3. Add an **AI Agent** node and connect it to the trigger.
4. Under the agent's **Chat Model**, add e.g. **OpenAI Chat Model** with your credential.
5. Under the agent's **Tool**, add **MCP Client Tool**:
   - Endpoint: `https://tariffsapi.com/api/v1/mcp`
   - Server Transport: `HTTP Streamable`
   - Authentication: `Bearer Auth` → credential `TariffsAPI`
   - Tools to Include: `All` (or `Selected`: ResolveTariffTool, GetByHtsTool, SearchProductsTool)
6. Optionally, set the agent's system message: "Use TariffsAPI tools for US tariff questions. Only state rates returned by the tools. TariffsAPI does not classify products."
7. Open the chat and ask: *"Using TariffsAPI, what duties apply to HTS 8471.30.0100 imported from China today?"*

## Import instead
Import [`../examples/n8n-tariffsapi-agent.json`](../examples/n8n-tariffsapi-agent.json) (Workflows → Import from File). Then reselect both credentials: the IDs in the file are placeholders.

## Troubleshooting
- 401: wrong key, or the Bearer credential has a "Bearer " prefix doubled
- 403 `PLAN_UPGRADE_REQUIRED`: upgrade to Developer or higher
- 429: rate or monthly limit (see README)
- No Server Transport field: your n8n version is older; update n8n
