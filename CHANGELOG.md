# Changelog

All notable changes to this plugin will be documented here.

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
