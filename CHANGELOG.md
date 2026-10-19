# Changelog

All notable changes to this plugin will be documented here.

## 1.0.1

- The plugin now runs Moxo's `@moxo/mcp-server` package locally with `npx` (pinned to major version 1) instead of connecting to `https://api.moxo.com/mcp-server` directly. Connecting directly failed in production because Moxo's servers keep each MCP session on one server, and Cursor's HTTP client does not send back the cookie that routes it there; the package does.
- Requires Node.js 20 or newer.

## 1.0.0 — initial release

- Added the `moxo` MCP server pointing at `https://api.moxo.com/mcp-server`.
- Auth uses a Moxo personal access token supplied by the user as a request header.
- Logo: Moxo's app icon.
