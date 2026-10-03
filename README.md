# algoum for AI assistants (MCP)

Let your AI assistant query US financial regulatory data from SEC EDGAR – insider trades
(Form 4/5), material events (8-K) and institutional holdings (13F) – through the
[Model Context Protocol](https://modelcontextprotocol.io). Ask in plain language, the assistant
looks up the data for you.

## What you can ask

Ask for what you want to know – the assistant picks the right tools for you.

### Insider trades (Form 4 and 5)

Buys and sells reported by officers, directors and 10% owners – for one company or across the whole
market – plus the signals derived from them: clusters of buyers and the buy/sell ratio of a
company's officers.

- "Which insiders bought NVIDIA shares in the last 7 days?"
- "Where did at least 3 insiders buy within the last 14 days?"
- "What is the officer buy/sell ratio for Tesla over the last 90 days?"
- "Show me the latest insider sales across the market."

### Material events (Form 8-K)

The events a company has to disclose – leadership changes, mergers, bankruptcies, cyber incidents,
earnings releases and more – as a searchable history or as the latest filings across all companies.

- "Which 8-K filings about cyber incidents came in today?"
- "What material events did Boeing report this year?"
- "Were there any bankruptcy filings in the last 24 hours?"

### Institutional holdings (Form 13F)

The quarterly portfolios of institutional investors and the aggregated picture per stock: how many
funds hold it, and how many opened, increased or reduced their position.

- "Who holds Apple in 2026-Q1, and what is the institutional sentiment?"
- "Show Bridgewater's portfolio for its latest reported quarter."
- "How many funds increased their Microsoft position last quarter?"
- "Which 13F disclosures came in this week?"

### Companies and filings

Ticker and company lookup, the original SEC filing behind every record, and how current each data
source is.

- "What is the CIK for Palantir?"
- "Give me the SEC filing behind that trade."
- "How fresh is the data?"

The assistant can also look up the [API description](https://api.algoum.de/v1/openapi.json) itself
whenever it needs more detail.

## Good to know

- Read-only access to published SEC data, taken straight from the official filings.
- Insider trades and material events become available within minutes of being filed.
- Timestamps are in US Eastern Time, exactly as the SEC reports them.
- 13F values are the holdings a fund reported for a quarter, so they describe positions rather than
  money flows.
- The assistant can ask for the current data freshness at any time.

Got a question, or an idea for data you would like to see? [Talk to us](https://algoum.de/contact) –
we are happy to hear from you.

The API is free to use. If it is useful to you, [a donation](https://algoum.de/about) helps us keep
the service running.

## Setup

### 1. Get your API key

Create a key in the dashboard of the [algoum portal](https://algoum.de/) – the same key you use for
the REST API. Create one key per app or machine, so each one stays under your control, and add or
revoke keys at any time.

Keep the key in an environment variable, or let your app prompt for it:

```sh
export ALGOUM_API_KEY='<your key>'
```

### 2. Check the connection

```sh
curl -s https://api.algoum.de/v1/mcp \
  -H "Authorization: Bearer $ALGOUM_API_KEY" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

### 3. Add the server to your assistant

Some assistants can install the server for you – look for **algoum** in the
[MCP Registry](https://registry.modelcontextprotocol.io/?q=algoum). With
[APM](#apm-agent-package-manager) two commands set it up for your assistants. Otherwise use the
values below.

|                |                                               |
|----------------|-----------------------------------------------|
| URL            | `https://api.algoum.de/v1/mcp`                |
| Transport      | Streamable HTTP                               |
| Authentication | Your API key as `Authorization: Bearer <key>` |

#### VS Code (GitHub Copilot)

Create `.vscode/mcp.json` in your project (or run **MCP: Open User Configuration** to use it in
all projects). VS Code asks for the key on first start and does not store it in the file:

```json
{
  "inputs": [
    { "type": "promptString", "id": "algoum-key", "description": "algoum API key", "password": true }
  ],
  "servers": {
    "algoum": {
      "type": "http",
      "url": "https://api.algoum.de/v1/mcp",
      "headers": { "Authorization": "Bearer ${input:algoum-key}" }
    }
  }
}
```

Switch the chat to agent mode and check in the tools list that the `algoum` tools are enabled.

#### IntelliJ IDEA and other JetBrains IDEs

Both options work in every JetBrains IDE. Use the one for the assistant you have installed.

**GitHub Copilot plugin**

1. Open **Settings → Tools → GitHub Copilot → Model Context Protocol (MCP)** and click
   **Configure**. This opens `mcp.json`.
2. Add the server:

   ```json
   {
     "servers": {
       "algoum": {
         "type": "http",
         "url": "https://api.algoum.de/v1/mcp",
         "headers": { "Authorization": "Bearer <your key>" }
       }
     }
   }
   ```

3. Open the Copilot chat, switch to agent mode and check that the `algoum` tools are listed.

**JetBrains AI Assistant**

1. Open **Settings → Tools → AI Assistant → Model Context Protocol (MCP)** and click **Add**.
2. Choose **As JSON** and paste:

   ```json
   {
     "mcpServers": {
       "algoum": {
         "url": "https://api.algoum.de/v1/mcp",
         "headers": { "Authorization": "Bearer <your key>" }
       }
     }
   }
   ```

3. Save and wait until the server shows as connected.

The key stays in the IDE's own configuration on your machine.

#### Claude Code

```sh
claude mcp add --transport http algoum https://api.algoum.de/v1/mcp \
  --header "Authorization: Bearer $ALGOUM_API_KEY"
```

Verify with `claude mcp list`.

#### Cursor

Create `.cursor/mcp.json` in your project (or `~/.cursor/mcp.json` for all projects). The key is
read from the environment variable you exported in step 1, so it does not end up in the file:

```json
{
  "mcpServers": {
    "algoum": {
      "url": "https://api.algoum.de/v1/mcp",
      "headers": { "Authorization": "Bearer ${env:ALGOUM_API_KEY}" }
    }
  }
}
```

Start Cursor from a shell in which `ALGOUM_API_KEY` is set, then check **Settings → MCP** that the
`algoum` server is connected.

#### GitHub Copilot CLI

Add the server to `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "algoum": {
      "type": "http",
      "url": "https://api.algoum.de/v1/mcp",
      "headers": { "Authorization": "Bearer <your key>" },
      "tools": ["*"]
    }
  }
}
```

Run `/mcp` in the CLI to see `algoum` connected. The file stays on your machine.

#### APM (Agent Package Manager)

[APM](https://github.com/microsoft/apm) installs the server for the assistant you choose. Run it in
your project. It reads the key from the environment variable you exported in step 1, so the key
does not end up in any file:

```sh
apm marketplace add fi4all/algoum_mcp
apm install algoum@algoum-marketplace --target vscode
```

Replace `vscode` with your assistant, for example `copilot`, `claude` or `cursor`. `apm targets`
lists all supported ones. Without `--target`, APM needs an existing assistant folder in the project.

## Links

- [algoum portal](https://algoum.de/)
- [API reference](https://api.algoum.de/v1/docs/)
- [MCP Registry entry](https://registry.modelcontextprotocol.io/?q=algoum)
- [Glama listing](https://glama.ai/mcp/connectors/de.algoum.api/algoum)
- [Contact us](https://algoum.de/contact) – questions, feedback and anything you would like to see next
