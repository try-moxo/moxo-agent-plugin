# Moxo

Plugin that connects AI agents in Cursor, Claude Code and Codex to [Moxo](https://moxo.com) through Moxo's hosted [Model Context Protocol](https://modelcontextprotocol.io/) server at `https://api.moxo.com/mcp-server`. Nothing runs on your machine.

Moxo is a workflow platform for business processes that run across teams and clients. With this plugin an agent can see which flows need attention, read where a flow stands, start a flow from a template, complete steps, and send reminders, all in the signed-in user's Moxo organization. It can also help you design a new process and build it as a Moxo template. Five skills turn the most common requests into a repeatable routine.

## Before you install

You need a **Moxo personal access token**. Create one in Moxo under **Settings → API keys → Create Personal Access Token**. It starts with `mxo_pat_`.

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

## Install in Codex

Codex reads your token from an environment variable, so the token never goes into Codex's config file.

1. Add your token to your shell profile (for example `~/.zshrc`), so every new terminal has it:

   ```bash
   export MOXO_API_TOKEN="mxo_pat_..."
   ```

2. Add this repository as a plugin marketplace, then install the plugin from it:

   ```bash
   codex plugin marketplace add try-moxo/moxo-agent-plugin
   codex plugin add moxo@moxo
   ```

3. Open a new terminal and run `codex mcp list`. The `moxo` row shows `MOXO_API_TOKEN` under **Bearer Token Env Var** and `Bearer token` under **Auth**. Then start Codex from that terminal.

The plugin brings the Moxo server and the skills. If you added Moxo by hand with `codex mcp add moxo` before, remove that first with `codex mcp remove moxo`. If you added this marketplace before version 1.3.0, run `codex plugin marketplace upgrade moxo` before `codex plugin add`.

## Other clients

Add Moxo's server to your client's MCP settings: `https://api.moxo.com/mcp-server`, over Streamable HTTP, with this header:

```
Authorization: Bearer <your personal access token>
```

If your client can't send a header, run Moxo's npm package [`@moxo/mcp-server`](https://www.npmjs.com/package/@moxo/mcp-server) instead (see **Notes**). The skills in `skills/` follow the [Agent Skills](https://agentskills.io) format, so a client that supports it can load them from this repository.

## Skills

| Skill | Use it for |
| --- | --- |
| `morning-triage` | "What needs me in Moxo?" Lists the steps waiting on you and the overdue flows, picks the one thing to do first, and offers reminders. |
| `start-flow-for-client` | "Start onboarding for Acme." Finds the person and the right template, fills every role and start field, and starts the flow only after you say yes to a summary. |
| `flow-status-report` | "How are the Acme flows doing this month?" A report for one client or one template over a date range: what started, finished, is stuck, and what happened. |
| `design-a-process` | "Help me set up vendor onboarding." Talks the process through in your words first (who, steps, decisions, deadlines, what happens when something is late or rejected), then shows how Moxo would run it and builds only after you say yes. |
| `build-a-template` | "Add a finance approval to the invoice template." Builds or changes a template as a draft, checks it, gives you a link and a test run, and publishes only when you say so. |

The first three skills build on the server's three prompts (`/show_my_attention`, `/start_flow_for`, `/flow_activity_summary`), which you can still run on their own.

## Build workflows

Describe a process and the agent builds it in Moxo with you. You can ask things like:

- "Help me design a client onboarding workflow."
- "Build an invoice approval: the requester submits, a manager approves, and anything over $10,000 also goes to finance."
- "Add a deadline to the vendor review step, then let me try it."

With a personal access token the agent builds as you: only what you could build in Moxo yourself, with the action types on your organization's plan. An org API key builds like an admin. A test run needs a personal access token.

Every change to a template goes to a draft, never straight to live work. The agent checks the draft after each change and fixes what it can, then gives you a link to open it in Moxo and offers a test run: you get the emails, marked [TEST], and the run stays out of live work and reports. It publishes only when you say yes.

The agent never asks for passwords, keys or tokens. When something needs a person in Moxo, such as connecting an app or adding a secret, it tells you what to do and gives you the link.

## MCP

Every client connects to the same server, `https://api.moxo.com/mcp-server`, over Streamable HTTP, and sends your token as `Authorization: Bearer <token>`. Each client reads its own configuration:

| Client | Configuration | Token comes from |
| --- | --- | --- |
| Cursor | `mcpServers` in `.cursor-plugin/plugin.json` | The plugin's `MOXO_API_TOKEN` variable, set in **Dashboard → Plugins → Configure** |
| Claude Code | `.mcp.json` (via `.claude-plugin/plugin.json`) | The plugin's `moxo_api_token` option, asked for when the plugin is enabled |
| Codex | `.codex-mcp.json` (via `.codex-plugin/plugin.json`) | The `MOXO_API_TOKEN` environment variable |
| Other clients | Your client's MCP settings | Your client's MCP settings (see above) |

This is the Cursor configuration:

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

The tool catalogue lives on Moxo's servers, so new tools appear without updating the plugin. Because the connection is plain HTTPS, it also works in cloud agents that can't run local programs.

## What the token can do

A personal access token acts as you. The agent can see and do what you can in Moxo, nothing more: someone who isn't an admin sees only their own flows and can't invite team members. Revoke the token at any time in **Settings → API keys**; the plugin stops working at once.

Write tools (`start_flow`, `complete_step`, `cancel_flow`, and the rest under **Actions** below) change real flows, and the build tools change and publish templates, so keep your client's tool approval on if you want to confirm each one.

## What agents can do

| Category | Tools |
| --- | --- |
| Your work | `list_attention_items`, `list_overdue_flows`, `get_dashboard_summary` |
| Flows | `list_flows`, `get_flow`, `list_flow_steps`, `get_flow_step`, `query_steps`, `list_flow_messages`, `get_flow_history`, `step_files` |
| Templates | `list_templates`, `get_template` |
| People | `list_external_users`, `get_external_user`, `list_team_members`, `get_team_member` |
| Search, reports and history | `search`, `get_report`, `query_audit_trail`, `get_org` |
| Actions | `start_flow`, `complete_step`, `act_on_step`, `add_step`, `upload_step_file`, `post_flow_message`, `send_reminder`, `manage_flow`, `cancel_flow`, `archive_flow`, `create_external_user`, `invite_team_member` |
| Learn Moxo | `search_moxo_docs`, `find_examples`, `get_organization_overview`, `get_action_type`, `get_guide` |
| Build templates | `create_template`, `change_template`, `check_template`, `get_template_outline`, `get_upload_link`, `start_test_run`, `discard_draft`, `publish_template` |
| Template setup | `manage_template_settings`, `manage_template_triggers`, `manage_agent`, `manage_data_table` |

The server also offers three prompts (`/show_my_attention`, `/flow_activity_summary`, `/start_flow_for`) and resources for flows and your attention queue. The hosted server is the source of truth for tool names and schemas; new tools appear without reinstalling the plugin.

## Notes

- **Self-hosted Moxo:** change the server URL to `https://<your-moxo-host>/mcp-server` after install.
- **No header support:** a client that can only start local programs can run [`@moxo/mcp-server`](https://www.npmjs.com/package/@moxo/mcp-server) instead: `npx -y @moxo/mcp-server@1` with `MOXO_API_TOKEN` set to your token. It needs Node.js 20 or newer.
- **Org API keys** (`mxo_sk_…`) also work as the token, but they act for the whole organization rather than a person, so tools that act as a person (`complete_step`, declining or reviewing with `act_on_step`, `post_flow_message`, `invite_team_member`, your attention queue, test runs, and agent, trigger and template settings) don't work with them, and neither does the `morning-triage` skill. Building templates works with an org key; it builds like an admin. An org key reaches only the tools its scopes allow. Use a personal access token unless you are setting up shared automation.
- Requests are limited to 600 per minute per token.

## Docs

- Moxo MCP server: https://developers.moxo.com/docs/mcp
- npm package: https://www.npmjs.com/package/@moxo/mcp-server
- Moxo REST API: https://developers.moxo.com/docs
- Server URL: https://api.moxo.com/mcp-server

Logo is Moxo's app icon.

## License

Apache-2.0
