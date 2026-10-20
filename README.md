# Moxo

Plugin that connects AI agents in Cursor, Claude Code and other [Agent Plugins](https://agent-plugins.org) clients to [Moxo](https://moxo.com) through Moxo's official [Model Context Protocol](https://modelcontextprotocol.io/) server, [`@moxo/mcp-server`](https://www.npmjs.com/package/@moxo/mcp-server), run locally on your machine.

Moxo is a workflow platform for business processes that run across teams and clients. With this plugin an agent can see which flows need attention, read where a flow stands, start a flow from a template, complete steps, and send reminders, all in the signed-in user's Moxo organization. Three skills turn the most common requests into a repeatable routine.

## Before you install

- **Node.js 20 or newer** on the machine running your agent, so that `npx` can download and run `@moxo/mcp-server` on first use.
- **A Moxo personal access token.** Create one in Moxo under **Settings → API keys → Create Personal Access Token**. It starts with `mxo_pat_`.

## Install in Cursor

1. Open **Cursor Settings → Plugins**.
2. Search for **Moxo**.
3. Click **Install**.
4. In **Dashboard → Plugins → Configure**, set **Moxo API token** to your personal access token.

Or run `/add-plugin moxo` in chat.

## Install in Claude Code

Add this repository as a plugin marketplace, then install the plugin from it:

```
/plugin marketplace add try-moxo/moxo-agent-plugin
/plugin install moxo@moxo
```

From a terminal, the same steps are `claude plugin marketplace add try-moxo/moxo-agent-plugin` and `claude plugin install moxo@moxo`.

Claude Code asks for your **Moxo API token** when the plugin is enabled. It keeps the token in the macOS Keychain, or in Claude Code's own credentials file on systems without one, never in `settings.json`. To change it later, run `/plugin configure moxo@moxo`. Uninstalling the plugin deletes the saved token.

## Install in other Agent Plugins clients

The repository root is an [Agent Plugins 1.0.0](https://agent-plugins.org/specification) package: `plugin.json`, `mcp.json` and `skills/`. Point your client at this repository the way it installs Agent Plugins.

Agent Plugins 1.0.0 has no way for a plugin to ask for a secret; it leaves sign-in to each client. `mcp.json` sets `MOXO_API_TOKEN` to `${MOXO_API_TOKEN}`, which Cursor fills in from its plugin settings. A client that follows the standard strictly passes that text through unchanged, and the server can't sign in. In that case, use the skills from this package and add the `moxo` server to your client's own MCP settings with your token.

## Skills

| Skill | Use it for |
| --- | --- |
| `morning-triage` | "What needs me in Moxo?" Lists the steps waiting on you and the overdue flows, picks the one thing to do first, and offers reminders. |
| `start-flow-for-client` | "Start onboarding for Acme." Finds the person and the right template, fills every role and start field, and starts the flow only after you say yes to a summary. |
| `flow-status-report` | "How are the Acme flows doing this month?" A report for one client or one template over a date range: what started, finished, is stuck, and what happened. |

The skills build on the server's three prompts (`/show_my_attention`, `/start_flow_for`, `/flow_activity_summary`), which you can still run on their own.

## MCP

Every client runs the same server with `npx`. Each client reads its own file:

| Client | File | Token comes from |
| --- | --- | --- |
| Cursor | `mcp.json` (via `.cursor-plugin/plugin.json`) | The plugin's `MOXO_API_TOKEN` variable, set in **Dashboard → Plugins → Configure** |
| Claude Code | `.mcp.json` (via `.claude-plugin/plugin.json`) | The plugin's `moxo_api_token` option, asked for when the plugin is enabled |
| Other Agent Plugins clients | `mcp.json` | Up to the client (see above) |

```json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/mcp.schema.json",
  "mcpServers": {
    "moxo": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@moxo/mcp-server@1"],
      "env": {
        "MOXO_API_TOKEN": "${MOXO_API_TOKEN}"
      }
    }
  }
}
```

`@moxo/mcp-server` is a small local bridge: your agent talks to it over stdio, and it forwards each request to Moxo at `https://api.moxo.com/mcp-server` over HTTPS. The tool catalogue lives on Moxo's servers, so new tools appear without updating the plugin. It is pinned to major version 1, so you get fixes but never a breaking change.

## What the token can do

A personal access token acts as you. The agent can see and do what you can in Moxo, nothing more: someone who isn't an admin sees only their own flows and can't invite team members. Revoke the token at any time in **Settings → API keys**; the plugin stops working at once.

Write tools (`start_flow`, `complete_step`, `cancel_flow`, and the rest under **Actions** below) change real flows, so keep your client's tool approval on if you want to confirm each one.

## What agents can do

| Category | Tools |
| --- | --- |
| Your work | `list_attention_items`, `list_overdue_flows`, `get_dashboard_summary` |
| Flows | `list_flows`, `get_flow`, `list_flow_steps`, `get_flow_step`, `query_steps`, `list_flow_messages` |
| Templates | `list_templates`, `get_template` |
| People | `list_external_users`, `get_external_user`, `list_team_members`, `get_team_member` |
| Search and history | `search`, `query_audit_trail`, `get_org` |
| Actions | `start_flow`, `complete_step`, `send_reminder`, `cancel_flow`, `archive_flow`, `create_external_user`, `invite_team_member` |

The server also offers three prompts (`/show_my_attention`, `/flow_activity_summary`, `/start_flow_for`) and live resources for flows and your attention queue. The hosted server is the source of truth for tool names and schemas; new tools appear without reinstalling the plugin.

## Notes

- **Self-hosted Moxo:** add `"MOXO_BASE_URL": "https://<your-moxo-host>"` to the server's `env` after install. The bridge adds `/mcp-server` itself.
- **Org API keys** (`mxo_sk_…`) also work if you add `"MOXO_ALLOW_ORG_KEY": "1"` to `env`, but they act for the whole organization rather than a person, so tools that act as a person (`complete_step`, `invite_team_member`, and your attention queue) don't work with them, and neither does the `morning-triage` skill. An org key reaches only the tools its scopes allow. Use a personal access token unless you are setting up shared automation.
- Requests are limited to 600 per minute per token.

## Docs

- Moxo MCP server: https://developers.moxo.com/docs/mcp
- npm package: https://www.npmjs.com/package/@moxo/mcp-server
- Moxo REST API: https://developers.moxo.com/docs
- Server URL: https://api.moxo.com/mcp-server

Logo is Moxo's app icon.

## License

Apache-2.0
