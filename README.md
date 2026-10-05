# OpenClawCash MCP Server

A crypto wallet for AI agents on Ethereum, Polygon, Base and Solana, as MCP tools. Your agent gets an
API key, never a private key, and every action is checked against the spending limits, allowlist and
testnet-only rules you set in [OpenClawCash](https://openclawcash.com) before anything is signed. It can
send, swap, bridge, trade on Polymarket and get paid through escrow.

This package is a thin stdio adapter over the OpenClawCash agent API at `https://openclawcash.com/api/agent/*`;
wallets, keys and policy enforcement stay on the server.

Canonical docs page: `https://openclawcash.com/mcp`

## What It Exposes

Wallet and profile tools:

- `skill_latest` (public install metadata; no API key required)
- `wallets_list`
- `wallet_get`
- `wallet_rename`
- `policies_list`
- `policy_get`
- `transactions_list`
- `balances_get`
- `supported_tokens_list`
- `user_tag_get`
- `user_tag_set`
- `wallet_create`

Trading and transfer tools:

- `transfer_send`
- `swap_quote`
- `swap_execute`
- `approve_token`

Escrow (formerly Get Paid) checkout tools:

- `checkout_payreq_create`
- `checkout_payreq_get`
- `checkout_escrow_get`
- `checkout_funding_confirm`
- `checkout_accept`
- `checkout_proof_submit`
- `checkout_dispute_open`
- `checkout_quick_pay`
- `checkout_swap_and_pay`
- `checkout_fund` (quick-pay first, swap fallback)
- `checkout_release`
- `checkout_refund`
- `checkout_cancel`
- `checkout_webhooks_list`
- `checkout_webhook_create`
- `checkout_webhook_update`
- `checkout_webhook_delete`

Webhooks deliver escrow events and `wallet.transaction.confirmed`. `"*"` covers escrow events only; the wallet event must be named in `eventTypes`.

Polymarket tools:

- `polymarket_market_resolve`
- `polymarket_market_search`
- `polymarket_order_limit`
- `polymarket_order_market`
- `polymarket_account`
- `polymarket_orders`
- `polymarket_cancel_order`
- `polymarket_clear_integration`
- `polymarket_activity`
- `polymarket_positions`
- `polymarket_redeemable`
- `polymarket_redeem`

Utility tool:

- `openclawcash-self-test`

It also exposes two lightweight MCP resources:

- `openclawcash://approval-modes`
- `openclawcash://quickstart`

## Run It

Direct from npm:

```bash
npx -y @openclawcash/mcp-server@0.1.27
```

From this folder during local development:

```bash
node openclawcash-mcp.mjs
```

If this package is installed globally or linked as a CLI:

```bash
openclawcash-mcp
```

Quick self-test:

```bash
npx -y @openclawcash/mcp-server@0.1.27 --self-test
```

## Configuration

Set one of these environment variables:

- `OPENCLAWCASH_AGENT_KEY`
- `AGENTWALLETAPI_KEY`

Most MCP tools require an OpenClawCash agent key because they call authenticated agent API routes. `skill_latest` is the only no-auth tool and calls `GET /api/public/agentwalletapi/skill/latest`.

Optional base URL override:

- `OPENCLAWCASH_BASE_URL`
- `AGENTWALLETAPI_URL`

The override must be `https://openclawcash.com` or an `https://<subdomain>.openclawcash.com` host (no port or credentials). Any other value makes every authenticated tool fail with an error instead of sending the agent key to that host. Redirects from the API are refused, so the key cannot follow one to another host.

Wallet export passphrase (needed only for `wallet_create`):

- `OPENCLAWCASH_EXPORT_PASSPHRASE` (at least 12 characters)

`wallet_create` reads the passphrase from this variable and never takes it as a tool argument, so it does not pass through the model or the chat transcript. Keep a copy somewhere safe: you need it to export a created wallet's key.

Importing an existing wallet by private key is not an MCP tool, for the same reason: a key passed as a tool argument would sit in the model context and the transcript. Import from the OpenClawCash dashboard ("Import Existing Wallet") or run the `agentwalletapi.sh import` CLI yourself.

The server also looks for env files in this order:

1. package `.env`
2. current working directory `.env.local`
3. current working directory `.env`

Example package `.env`:

```env
OPENCLAWCASH_AGENT_KEY=occ_your_api_key_here
OPENCLAWCASH_BASE_URL=https://openclawcash.com
```

## OpenClaw Example

You can print a config snippet with:

```bash
npx -y @openclawcash/mcp-server@0.1.27 --print-openclaw-config
```

Typical shape:

```json
{
  "mcpServers": {
    "openclawcash": {
      "command": "npx",
      "args": ["-y", "@openclawcash/mcp-server@0.1.27"],
      "env": {
        "OPENCLAWCASH_AGENT_KEY": "occ_your_api_key_here",
        "OPENCLAWCASH_BASE_URL": "https://openclawcash.com"
      }
    }
  }
}
```

If OpenClaw expects the same MCP structure in a different config location, keep the same `command`, `args`, and `env` values and adapt only the outer wrapper required by that client.

## Claude Desktop Example

You can print a config snippet with:

```bash
npx -y @openclawcash/mcp-server@0.1.27 --print-claude-config
```

Typical config shape:

```json
{
  "mcpServers": {
    "openclawcash": {
      "command": "npx",
      "args": ["-y", "@openclawcash/mcp-server@0.1.27"],
      "env": {
        "OPENCLAWCASH_AGENT_KEY": "occ_your_api_key_here",
        "OPENCLAWCASH_BASE_URL": "https://openclawcash.com"
      }
    }
  }
}
```

## OpenAI Codex Example

If your OpenAI Codex environment supports MCP server configuration, use the same local stdio command pattern:

```json
{
  "mcpServers": {
    "openclawcash": {
      "command": "npx",
      "args": ["-y", "@openclawcash/mcp-server@0.1.27"],
      "env": {
        "OPENCLAWCASH_AGENT_KEY": "occ_your_api_key_here",
        "OPENCLAWCASH_BASE_URL": "https://openclawcash.com"
      }
    }
  }
}
```

If your Codex setup expects a different config file location or wrapper schema, keep the same `command`, `args`, and `env` values and adapt only the surrounding structure required by that client.

## Cursor Example

You can print a config snippet with:

```bash
npx -y @openclawcash/mcp-server@0.1.27 --print-cursor-config
```

If your Cursor build supports MCP servers, the config shape is the same idea: run the server as a local stdio command and pass the API key in env.

Typical shape:

```json
{
  "mcpServers": {
    "openclawcash": {
      "command": "npx",
      "args": ["-y", "@openclawcash/mcp-server@0.1.27"],
      "env": {
        "OPENCLAWCASH_AGENT_KEY": "occ_your_api_key_here",
        "OPENCLAWCASH_BASE_URL": "https://openclawcash.com"
      }
    }
  }
}
```

If your Cursor build expects the config in a different file location or format, keep the same command, args, and env values and adapt only the wrapper structure.

## VS Code Example

You can print a config snippet with:

```bash
npx -y @openclawcash/mcp-server@0.1.27 --print-vscode-config
```

If your VS Code AI extension or MCP-compatible tooling supports local stdio MCP servers, use the same command pattern:

```json
{
  "mcpServers": {
    "openclawcash": {
      "command": "npx",
      "args": ["-y", "@openclawcash/mcp-server@0.1.27"],
      "env": {
        "OPENCLAWCASH_AGENT_KEY": "occ_your_api_key_here",
        "OPENCLAWCASH_BASE_URL": "https://openclawcash.com"
      }
    }
  }
}
```

If your VS Code setup uses a different config file location or wrapper format, keep the same `command`, `args`, and `env` values and adapt only the surrounding JSON structure required by that extension.

## Verify Before IDE Setup

Use these commands before wiring the server into any IDE:

```bash
npx -y @openclawcash/mcp-server@0.1.27 --help
npx -y @openclawcash/mcp-server@0.1.27 --self-test
npx -y @openclawcash/mcp-server@0.1.27 --print-openclaw-config
npx -y @openclawcash/mcp-server@0.1.27 --print-claude-config
npx -y @openclawcash/mcp-server@0.1.27 --print-cursor-config
npx -y @openclawcash/mcp-server@0.1.27 --print-vscode-config
```

What `--self-test` checks:

- the server file runs
- the MCP metadata is valid enough for initialization
- the resolved base URL is correct
- whether an agent key is present in env or env files

## Approval Model

Write tools are high-risk. The intended agent behavior is:

- Ask once at the start of the first write-intent:
  - `confirm_each_write`
  - `operate_on_my_behalf`
- If the user chooses `operate_on_my_behalf`, the agent should stop re-asking for each later transfer in the same session and only ask for missing execution details.

The MCP server itself does not hold approval memory. The MCP client or agent runtime should remember the selected mode for the session.

## Notes

- `swap_quote` is read-only, but still requires an agent key.
- `supported_tokens_list` is read-only, but still requires an agent key.
- `skill_latest` is read-only and does not require an agent key.
- `transfer_send`, `swap_execute`, `approve_token`, and `wallet_create` are write tools.
- `wallet_rename` and `transfer_send` select the wallet by `walletId` only and reject `walletLabel` or `walletAddress`. No write tool accepts `walletLabel`: labels are user-controlled text.
- `polymarket_redeemable` is read-only and lists redeemable tokenIds; use this first for targeted redeem.
- `polymarket_redeem` is a write tool. For `redeem all` (no tokenId), the API processes in chunks and returns `hasMoreRedeemable`; call again until it becomes `false`.
- `transfer_send` is for normal wallet transfers. For checkout escrow funding, use checkout tools:
  - `checkout_quick_pay` for direct settlement funding
  - `checkout_swap_and_pay` for asset mismatch funding
  - `checkout_funding_confirm` to confirm external/manual funding transactions
- For `wallet_get`, `transactions_list`, `supported_tokens_list`, and `swap_quote`, pass at most one wallet selector when using `walletId`, `walletLabel`, or `walletAddress`.
- The server uses stdio transport and is intended for MCP-compatible desktop or agent clients.

## Standalone Packaging Notes

- This is its own repo with its own [package.json](package.json), separate from the main web app repo.
- The CLI name is `openclawcash-mcp`.
- The runtime does not depend on any other repo's paths.
- The intended public install path is `npx -y @openclawcash/mcp-server@0.1.27`.

## Release Checklist

Before publishing a new MCP package release:

1. Update the version in [package.json](package.json), `package-lock.json`, `SERVER_VERSION` in `openclawcash-mcp.mjs`, and every pinned `@openclawcash/mcp-server@<version>` in this README.
2. Run the package self-test:
   ```bash
   node openclawcash-mcp.mjs --self-test
   ```
3. Verify the config helpers:
   ```bash
   node openclawcash-mcp.mjs --print-openclaw-config
   node openclawcash-mcp.mjs --print-claude-config
   node openclawcash-mcp.mjs --print-cursor-config
   node openclawcash-mcp.mjs --print-vscode-config
   ```
4. Confirm the package contents:
   ```bash
   npm pack --dry-run
   ```
5. Review that no secrets are present:
   - no `.env`
   - no local credentials
   - no private test data
6. Commit, then push a matching version tag. The `publish` GitHub Actions workflow re-runs the checks and
   publishes to npm with provenance through npm trusted publishing (no npm token is stored):
   ```bash
   git tag v<version> && git push origin main v<version>
   ```
   The workflow refuses to publish if the tag does not match `package.json` and `SERVER_VERSION`.

If you want to test the package locally before publishing:

```bash
npm pack
```

Then install the generated tarball in a separate test directory and verify the CLI runs there without repo context.
