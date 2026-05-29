# Noelclaw — Autonomous Crypto Agent Platform

Noelclaw is a multi-agent crypto platform that turns plain-English instructions into on-chain actions. It combines a 38-tool MCP server, an NL-to-automation engine, a multi-agent swarm, and a persistent vault — all running on Base with USDC via x402.

## Core Capabilities

**Trading & DeFi (via 0x on Base)**
- Portfolio: `get_portfolio` — get all token balances and total USD value for the MCP wallet
- Swap preview: `estimate_swap` — see expected output amount and price impact before executing
- Token swaps: `swap_tokens` — execute any ERC-20↔ERC-20 swap on Base through the user's custodial MCP wallet
- Token transfers: `send_token` — send ETH, USDC, USDT, DAI to any address
- Wallet resolution: `get_wallet_address` — look up or create a user's encrypted on-chain wallet

**Natural Language Automations**
- `create_automation` — parse plain English into a structured trigger + action pair, then run it on a cron
  - **Triggers**: `schedule` (interval), `price_drop_%`, `price_rise_%`, `price_below`, `price_above`, `dominance_below`, `dominance_above`
  - **Actions**: `swap`, `send`, `alert` (Telegram)
  - **Limits**: `maxRuns`, `maxSpendUsd`, expiry
  - Examples: *"Buy $50 of ETH every day, stop after $500"*, *"If ETH drops 10%, buy $100 USDC→ETH"*, *"Alert me when BTC dominance drops below 50%"*
- `list_automations` / `pause_automation` / `delete_automation` / `get_automation_runs` — see per-run history with status, tx hash, and error details

**Agent Swarm**
Five specialized sub-agents run autonomously when the swarm is active:
- **Market Monitor** — tracks live prices, detects volume spikes and resistance breaks
- **Sentiment Tracker** — scans on-chain signals and social data
- **Workflow Executor** — fires scheduled automations and DCA strategies
- **Memory Manager** — compresses and organises shared swarm memory
- **Risk Verifier** — gates every action through a configurable risk score threshold

Tools: `start_swarm`, `stop_swarm`, `get_swarm_status`, `get_swarm_memory`, `write_swarm_memory`, `get_execution_scores`

**Market Intelligence**
- `get_market_data` — live prices, trending coins, top-20 market cap via Bankr LLM API
- `get_token_data` — deep token analysis (price, volume, sentiment, on-chain activity)
- `ask_noel` — general crypto Q&A and reasoning, powered by Bankr LLM

**MiroShark Simulation**
- `miroshark_simulate` — run a multi-agent market simulation (bull/bear/neutral scenario modelling)
- `miroshark_status` — poll simulation progress and get final results with agent activity breakdown
- `miroshark_stop` — stop a running simulation

**Framework (Agent Orchestration)**
- `create_task_packet` / `list_task_packets` — define and track structured agent task bundles
- `list_playbooks` / `run_playbook` — run predefined multi-step agent workflows
- `get_noel_ledger` — view the agent action ledger (all tool calls, results, timestamps)
- `get_sentinel_rules` — retrieve active risk rules that gate autonomous actions

**Vault (Persistent Agent Memory)**
- `vault_save` / `vault_read` / `vault_search` / `vault_list` — store and retrieve agent outputs, strategies, research notes
- `vault_history` / `vault_diff` / `vault_export` — version history and diff between entries

**Other**
- `post_tweet` — post to X/Twitter with optional humanization
- `humanize_text` — rewrite AI-generated text to sound natural
- `set_telegram` — link a Telegram chat for alert delivery

## Access

**MCP Server**

```bash
npx -y @noelclaw/mcp
```

Add to Claude Desktop or any MCP-compatible client:
```json
{
  "mcpServers": {
    "noelclaw": {
      "command": "npx",
      "args": ["-y", "@noelclaw/mcp"],
      "env": { "NOELCLAW_API_KEY": "your_key_here" }
    }
  }
}
```

Get your API key at **noelclaw.com/api-keys** (7 free keys per account).

**HTTP API**
Base URL: `https://api.noelclaw.com`

| Auth Method | Header |
|---|---|
| Session token | `Authorization: Bearer <token>` |
| x402 micropayment | Send USDC on Base → retry with `X-Payment: {"txHash":"...","requestId":"..."}` |
| Wallet signature | `X-Wallet-Address` + `X-Wallet-Signature` + `X-Wallet-Timestamp` |

Free tools (`get_market_data`, `ask_noel`) pass through with no auth required.

## Key Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/mcp/market` | GET | Live prices, trending, top-20 |
| `/mcp/chat` | POST | Ask Noel any crypto question |
| `/automations/create` | POST | Create automation from plain English |
| `/automations/list` | GET | List user's automations |
| `/automations/pause` | POST | Pause or resume an automation |
| `/automations/delete` | POST | Delete an automation |
| `/swarm/start` | POST | Start the agent swarm |
| `/swarm/stop` | POST | Stop the swarm |
| `/swarm/status` | GET | Current swarm status + memory |
| `/swarm/scores` | GET | Agent execution scores |
| `/mcp/defi/swap` | POST | Get 0x swap quote |
| `/mcp/defi/send` | POST | Get send tx data |

## Integration Pattern

Noelclaw works as an **execution + intelligence layer** alongside Bankr:
- Use `get_market_data` / `ask_noel` for research (free, no auth)
- Use `create_automation` to set recurring DCA/alert strategies in plain English
- Use `swap_tokens` / `send_token` for immediate on-chain execution
- Use `start_swarm` for autonomous 24/7 market monitoring and execution
- Use `vault_save` to persist agent research and strategy outputs across sessions

All swap/send operations go through the user's personal encrypted MCP wallet on Base. The swarm's risk-verifier gates every autonomous action against a configurable threshold before execution.

## Tech Stack

- **Bankr LLM API** (`llm.bankr.bot`) — all agent reasoning and market intelligence
- **x402 protocol** — native USDC micropayment support on Base
- **MCP (Model Context Protocol)** — 38 tools, stdio transport, v2.1.0
- **Convex** — real-time backend, cron automation engine, swarm coordinator
- **0x Protocol v2** — on-chain swap execution on Base
- **Base mainnet** — all token operations and payments
