# AI Note — Claude Code plugin

Claude Code plugin for [AI Note](https://docs.ainote.dev): tasks, notes,
session handoffs, and team dev docs for coding agents, served over a remote
MCP server — no local install required.

## Install (Claude Code)

```bash
claude plugin marketplace add seunghan91/ainote-plugins
claude plugin install ainote@ainote
```

Get an API key (either way):

- In the agent: call the `signup_and_get_key` / `login_and_get_key` MCP tools.
- On the CLI: `npx @ainote/mcp login` (device flow).

Then export it as `AINOTE_API_KEY` so the plugin's MCP config can send it.

## What you get

- **ainote-handoff** skill — save a self-contained session handoff and resume
  it from any device or session (`handoff_save` / `handoff_list` / `handoff_get`).
- **ainote-tasks** skill — manage tasks and categories from the agent.
- **ainote-devdocs** skill — centrally manage CLAUDE.md / Cursor rules and
  other dev docs across machines and teammates.
- 50+ MCP tools total — see the [tool reference](https://docs.ainote.dev/reference/).

## Other agent clients

The skills follow the open [Agent Skills](https://agentskills.io) format —
copy `plugins/ainote/skills/<name>/` into your client's skills directory.
For MCP, point your client at `https://api.ainote.dev/api/mcp`
(header `Authorization: McpKey <your key>`) or run the stdio bridge
`npx @ainote/mcp`. Client-specific guides: https://docs.ainote.dev/mcp/overview

## License

MIT
