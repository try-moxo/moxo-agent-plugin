# Changelog

All notable changes to this plugin will be documented here.

## 1.4.0

- build-a-template sends `draftRevision` as `baseVersion`; `version` is the published version number.

- `manage_template_versions`: list a template's published versions and restore an earlier one into the draft (running flows keep their version).

- The internal-user tools use product words: `list_internal_users`, `get_internal_user`, `invite_internal_user` (were `list_team_members`, `get_team_member`, `invite_team_member`).

- Two new skills for building workflows. `design-a-process` talks a process through in plain words (who is involved, the steps, decisions, deadlines, what happens when something is late or rejected, and which systems it touches), maps it onto what Moxo can do, and shows a short design to approve. `build-a-template` builds or changes the template as a draft in small, checked steps, gives you a link and a test run, and publishes only on your yes.
- `start-flow-for-client` points to the new skills for building templates, and to a test run when you only want to try a template.
- README: a **Build workflows** section, and the new learn, build and template setup tools.
- Build with a personal access token (as you) or an org API key (like an admin). A test run needs a personal access token.
- README lists every tool the server offers, including `act_on_step`, `add_step`, `manage_flow`, `post_flow_message`, `upload_step_file`, `step_files`, `get_flow_history` and `get_report`.

## 1.3.0

- Codex: install the plugin with `codex plugin marketplace add try-moxo/moxo-agent-plugin`, then `codex plugin add moxo@moxo`. It brings the Moxo server and the three skills, and reads the token from the `MOXO_API_TOKEN` environment variable. New files: `.agents/plugins/marketplace.json`, `.codex-plugin/plugin.json` and `.codex-mcp.json`.
- Removed the root `plugin.json` and `mcp.json` (the Agent Plugins package). When a root `plugin.json` is present, Codex takes the server from the root `mcp.json` and ignores the one in `.codex-plugin/plugin.json`, and the root file can't carry a token. The root package couldn't sign in anywhere for the same reason: the standard has no way to supply a token. Cursor and Claude Code don't read these files and are unchanged.

## 1.2.0

- The plugin connects straight to Moxo's hosted server at `https://api.moxo.com/mcp-server` over HTTP again, with the token sent as `Authorization: Bearer <token>`. Moxo's servers now answer a connection on any server, so the problem that 1.0.1 worked around is gone. Node.js is no longer needed, and the plugin works in cloud agents.
- Cursor: the server is configured in `.cursor-plugin/plugin.json`, the same way 1.0.0 did it. Set the token as before in **Dashboard → Plugins → Configure**.
- Claude Code: `.mcp.json` sends the `moxo_api_token` option as the header. Nothing changes for people who already set it.
- Agent Plugins: `mcp.json` names the server and carries no token, as the standard requires. Add the header in your client's MCP settings.
- `@moxo/mcp-server` still works for clients that can only run local programs.

## 1.1.0

- The repository is now `try-moxo/moxo-agent-plugin` (was `moxo-cursor-plugin`), since it serves more than Cursor. GitHub redirects the old address.
- Claude Code support: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` and `.mcp.json`. Install with `/plugin marketplace add try-moxo/moxo-agent-plugin`, then `/plugin install moxo@moxo`. Claude Code asks for the Moxo API token when the plugin is enabled and keeps it out of `settings.json` (in the macOS Keychain, or Claude Code's credentials file elsewhere).
- Agent Plugins 1.0.0 support: a root `plugin.json`, and `mcp.json` now declares the Agent Plugins schema. Cursor reads the same `mcp.json` as before.
- Three skills: `morning-triage`, `start-flow-for-client` and `flow-status-report`. Cursor and Claude Code load them from `skills/`.
- Same server as 1.0.1: `npx -y @moxo/mcp-server@1`.

## 1.0.1

- The plugin now runs Moxo's `@moxo/mcp-server` package locally with `npx` (pinned to major version 1) instead of connecting to `https://api.moxo.com/mcp-server` directly. Connecting directly failed in production because Moxo's servers keep each MCP session on one server, and Cursor's HTTP client does not send back the cookie that routes it there; the package does.
- Requires Node.js 20 or newer.

## 1.0.0 — initial release

- Added the `moxo` MCP server pointing at `https://api.moxo.com/mcp-server`.
- Auth uses a Moxo personal access token supplied by the user as a request header.
- Logo: Moxo's app icon.
