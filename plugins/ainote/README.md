# ainote plugin

Claude Code plugin for [ainote](https://docs.ainote.dev) — tasks, notes, dev-docs, and session handoffs for coding agents, backed by a remote MCP server.

Includes three skills:
- `ainote-handoff` — save/restore session handoff notes for cross-device or cross-session continuation.
- `ainote-tasks` — task and category CRUD.
- `ainote-devdocs` — team dev-doc CRUD, including centrally distributing `CLAUDE.md` / Cursor rules / memory files across machines.

## Install

### Path 1 — plugin install (recommended)

Add the marketplace and install the plugin from within Claude Code:

```
/plugin marketplace add seunghan91/ainote-plugins
/plugin install ainote@ainote
```

This registers the bundled `.mcp.json` (remote HTTP transport) and the three skills above automatically.

### Path 2 — manual MCP config

If you're not using the plugin system, register the MCP server directly. Add to `~/.claude.json` (user-level) or a project's `.mcp.json`:

```json
{
  "mcpServers": {
    "ainote": {
      "type": "http",
      "url": "https://api.ainote.dev/api/mcp",
      "headers": {
        "Authorization": "McpKey YOUR_KEY_HERE"
      }
    }
  }
}
```

The `"type": "http"` field is required — omitting it silently fails schema validation and can prevent the entire `mcpServers` block (including unrelated servers) from loading.

No key yet? Register without the `headers` field first, then ask your agent to run `signup_and_get_key` or `login_and_get_key` to obtain one.

## Alternative transport: stdio (local)

To keep MCP traffic from leaving your machine via a locally-wrapped process (it still calls `api.ainote.dev` over HTTPS under the hood):

```bash
npm install -g @ainote/mcp
```

```json
{
  "mcpServers": {
    "ainote": {
      "command": "npx",
      "args": ["-y", "@ainote/mcp"],
      "env": {
        "AINOTE_API_KEY": "YOUR_KEY_HERE"
      }
    }
  }
}
```

## Getting an MCP key

Any of:
- Ask your agent to call `signup_and_get_key` (new account) or `login_and_get_key` (existing account) — no auth header needed for these two calls.
- Run `npx @ainote/mcp login` (RFC 8628 device-authorization flow).
- Visit `https://app.ainote.dev` → Settings → MCP keys.

Set the result as `AINOTE_API_KEY` (stdio transport) or in the `Authorization: McpKey <key>` header (HTTP transport).

## Links

- Docs: https://docs.ainote.dev
- Full tool catalog: https://docs.ainote.dev/reference/
- npm package: https://www.npmjs.com/package/@ainote/mcp
- Repository: https://github.com/seunghan91/ainote
