# Moxo

Cursor plugin that connects agents to [Moxo](https://moxo.com) through Moxo's official remote [Model Context Protocol](https://modelcontextprotocol.io/) server.

Moxo is a workflow platform for business processes that run across teams and clients. With this plugin an agent can see which flows need attention, read where a flow stands, start a flow from a template, complete steps, and send reminders, all in the signed-in user's Moxo organization.

## Install

1. Open **Cursor Settings → Plugins**.
2. Search for **Moxo**.
3. Click **Install**, then set your Moxo API token (below).

Or run `/add-plugin moxo` in chat.

## MCP

```json
{
  "mcpServers": {
    "moxo": {
      "type": "http",
      "url": "https://api.moxo.com/mcp-server",
      "headers": {
        "Authorization": "Bearer ${MOXO_API_TOKEN}"
      }
    }
  }
}
```

Auth is a Moxo **personal access token** sent as a bearer token. Create one in Moxo under **Settings → API keys → Create Personal Access Token** (it starts with `mxo_pat_`), then set it in **Dashboard → Plugins → Configure**.

## What the token can do

A personal access token acts as you. The agent can see and do what you can in Moxo, nothing more: someone who isn't an admin sees only their own flows and can't invite team members. Revoke the token at any time in **Settings → API keys**; the plugin stops working at once.

Write tools (`start_flow`, `complete_step`, `cancel_flow`, and the rest under **Actions** below) change real flows, so keep Cursor's tool approval on if you want to confirm each one.

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

- **Self-hosted Moxo:** change `url` in `mcp.json` to `https://<your-moxo-host>/mcp-server` after install.
- **Org API keys** (`mxo_sk_…`) also work but act for the whole organization rather than a person, so tools that act as a person (`complete_step`, `invite_team_member`, and your attention queue) don't work with them. An org key reaches only the tools its scopes allow. Use a personal access token unless you are setting up shared automation.
- Requests are limited to 600 per minute per token.

## Docs

- Moxo MCP server: https://developers.moxo.com/docs/mcp
- Moxo REST API: https://developers.moxo.com/docs
- Server URL: https://api.moxo.com/mcp-server

Logo is Moxo's app icon.

## License

Apache-2.0
