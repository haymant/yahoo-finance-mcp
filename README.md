[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/alex2yang97-yahoo-finance-mcp-badge.png)](https://mseep.ai/app/alex2yang97-yahoo-finance-mcp)

# Yahoo Finance MCP Server

<div align="right">
  <a href="README.md">English</a> | <a href="README.zh.md">中文</a>
</div>

This is a Model Context Protocol (MCP) server that provides comprehensive financial data from Yahoo Finance. It allows you to retrieve detailed information about stocks, including historical prices, company information, financial statements, options data, and market news.

[![smithery badge](https://smithery.ai/badge/@Alex2Yang97/yahoo-finance-mcp)](https://smithery.ai/server/@Alex2Yang97/yahoo-finance-mcp)

## Demo

![MCP Demo](assets/demo.gif)

## MCP Tools

The server exposes the following tools through the Model Context Protocol:

### Stock Information

| Tool | Description |
|------|-------------|
| `get_historical_stock_prices` | Get historical OHLCV data for a stock with customizable period and interval |
| `get_stock_info` | Get comprehensive stock data including price, metrics, and company details |
| `get_yahoo_finance_news` | Get latest news articles for a stock |
| `get_stock_actions` | Get stock dividends and splits history |

### Financial Statements

| Tool | Description |
|------|-------------|
| `get_financial_statement` | Get income statement, balance sheet, or cash flow statement (annual/quarterly) |
| `get_holder_info` | Get major holders, institutional holders, mutual funds, or insider transactions |

### Options Data

| Tool | Description |
|------|-------------|
| `get_option_expiration_dates` | Get available options expiration dates |
| `get_option_chain` | Get options chain for a specific expiration date and type (calls/puts) |

### Analyst Information

| Tool | Description |
|------|-------------|
| `get_recommendations` | Get analyst recommendations or upgrades/downgrades history |

## Real-World Use Cases

With this MCP server, you can use Claude to:

### Stock Analysis

- **Price Analysis**: "Show me the historical stock prices for AAPL over the last 6 months with daily intervals."
- **Financial Health**: "Get the quarterly balance sheet for Microsoft."
- **Performance Metrics**: "What are the key financial metrics for Tesla from the stock info?"
- **Trend Analysis**: "Compare the quarterly income statements of Amazon and Google."
- **Cash Flow Analysis**: "Show me the annual cash flow statement for NVIDIA."

### Market Research

- **News Analysis**: "Get the latest news articles about Meta Platforms."
- **Institutional Activity**: "Show me the institutional holders of Apple stock."
- **Insider Trading**: "What are the recent insider transactions for Tesla?"
- **Options Analysis**: "Get the options chain for SPY with expiration date 2024-06-21 for calls."
- **Analyst Coverage**: "What are the analyst recommendations for Amazon over the last 3 months?"

### Investment Research

- "Create a comprehensive analysis of Microsoft's financial health using their latest quarterly financial statements."
- "Compare the dividend history and stock splits of Coca-Cola and PepsiCo."
- "Analyze the institutional ownership changes in Tesla over the past year."
- "Generate a report on the options market activity for Apple stock with expiration in 30 days."
- "Summarize the latest analyst upgrades and downgrades in the tech sector over the last 6 months."

## Requirements

- Python 3.11 or higher
- Dependencies as listed in `pyproject.toml`, including:
  - mcp
  - yfinance
  - pandas
  - pydantic
  - and other packages for data processing

## Setup

### Recommended: run with `uvx`

Run the server directly from the repository without creating a local virtual environment:

```bash
uvx --from git+https://github.com/Alex2Yang97/yahoo-finance-mcp yahoo-finance-mcp
```

### Local development

1. Clone this repository:
   ```bash
   git clone https://github.com/Alex2Yang97/yahoo-finance-mcp.git
   cd yahoo-finance-mcp
   ```

2. Create and activate a virtual environment and install dependencies:
   ```bash
   uv venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   uv pip install -e .
   ```

## Running on v0.app / Vercel (Streamable HTTP)

In addition to the local stdio transport, this server can run as a remote MCP
server over **Streamable HTTP**, which is what [v0.app](https://v0.app) and
Vercel use. This makes the server reachable over the network so MCP clients
(including the MCP Inspector) can connect to a hosted URL.

### How it works

- `backend/main.py` re-exports the same `yfinance` FastMCP server, but exposes it
  as an ASGI app via `FastMCP(..., stateless_http=True).streamable_http_app()`.
  `stateless_http=True` is required for serverless platforms because each
  request must be self-contained (no in-memory session that survives between
  invocations or cold starts).
- `vercel.json` declares a Python service using Vercel's `experimentalServices`
  API. The service mounts `backend/main.py` under the `/api` route prefix.
  Vercel strips that prefix before forwarding to the app, and FastMCP serves the
  Streamable HTTP transport at `/mcp`, so the **public MCP endpoint** is:

  ```
  https://<your-deployment>.vercel.app/api/mcp
  ```

### Deploy

1. Open / fork this project in [v0.app](https://v0.app), or import the repo into
   Vercel.
2. Deploy. If you import directly into Vercel and see a 404, set the
   **Framework Preset** to `Services` (Settings → Build and Deployment).
3. After deploying, your MCP endpoint is available at `https://<your-deployment>.vercel.app/api/mcp`.

## Connecting from the MCP Inspector

You can test the hosted server with the official
[MCP Inspector](https://github.com/modelcontextprotocol/inspector).

1. Run the Inspector (no install required):

   ```bash
   npx @modelcontextprotocol/inspector
   ```

   This opens the Inspector UI in your browser.

2. In the Inspector UI, set:
   - **Transport Type**: `Streamable HTTP`
   - **URL**: `https://<your-deployment>.vercel.app/api/mcp`

   (When testing locally with `vercel dev`, use `http://localhost:3000/api/mcp`.)

3. Click **Connect**, then open the **Tools** tab and click **List Tools**. You
   should see all nine tools (`get_historical_stock_prices`, `get_stock_info`,
   `get_yahoo_finance_news`, `get_stock_actions`, `get_financial_statement`,
   `get_holder_info`, `get_option_expiration_dates`, `get_option_chain`,
   `get_recommendations`). Select one (e.g. `get_stock_info` with
   `ticker = AAPL`) and click **Run Tool** to verify a response.

You can also smoke-test the endpoint from the command line:

```bash
curl -X POST https://<your-deployment>.vercel.app/api/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

## Usage

### Quick Start

Run the packaged entrypoint with:

```bash
uvx --from git+https://github.com/Alex2Yang97/yahoo-finance-mcp yahoo-finance-mcp
```

For local changes in this checkout, use:

```bash
uvx --from . yahoo-finance-mcp
```

### Development Mode

If you are working inside a local clone and want to run the source tree directly:

```bash
uv run server.py
```

### Integration with Claude for Desktop

To integrate this server with Claude for Desktop:

1. Install Claude for Desktop to your local machine.
2. Install VS Code to your local machine. Then run the following command to open the `claude_desktop_config.json` file:
   - MacOS: `code ~/Library/Application\ Support/Claude/claude_desktop_config.json`
   - Windows: `code $env:AppData\Claude\claude_desktop_config.json`

3. Edit the Claude for Desktop config file, located at:
   - macOS: 
     ```json
     {
       "mcpServers": {
         "yfinance": {
           "command": "uvx",
           "args": [
             "--from",
             "git+https://github.com/Alex2Yang97/yahoo-finance-mcp",
             "yahoo-finance-mcp"
           ]
         }
       }
     }
     ```
   - Windows:
     ```json
     {
       "mcpServers": {
         "yfinance": {
           "command": "uvx",
           "args": [
             "--from",
             "git+https://github.com/Alex2Yang97/yahoo-finance-mcp",
             "yahoo-finance-mcp"
           ]
         }
       }
     }
     ```

   - **Note**: You may need to put the full path to the uv executable in the command field. You can get this by running `which uv` on MacOS/Linux or `where uv` on Windows.

4. Restart Claude for Desktop

## License

MIT
