# QuickStatus skill

A website check is up when the HTTP status is below 500. An API check is up on 200 unless another code is set. DNS is up when the lookup succeeds. TCP is up when the connection succeeds, or when it fails if that was the expected outcome. The same workspace publishes a status page, opens incidents, and schedules maintenance. [QuickStatus](https://quickstatus.dev).

The server is `https://api.quickstatus.dev/mcp`. It uses a workspace API key (`SP_API_KEY`), not OAuth. Create the key at https://quickstatus.dev/app/api-keys. Setup steps: https://quickstatus.dev/docs/mcp.

## Install the skill

```bash
npx skills add QuickStatusHQ/skills --skill uptime-status-page -a claude-code -a codex -g -y
```

Then wire MCP (reads `SP_API_KEY` from the environment when tools run):

```bash
bash ~/.agents/skills/uptime-status-page/scripts/wire-mcp.sh codex
```

Use `cursor`, `claude`, or `all` instead of `codex`. Catalog: https://www.skills.sh/quickstatushq/skills/uptime-status-page

## Cursor Marketplace

This repo is also a Cursor plugin. After you install it, set `SP_API_KEY` under Plugins → Configure. The plugin does not contain a key.

Local load, before the marketplace listing is approved:

```bash
mkdir -p ~/.cursor/plugins/local
ln -s "$(pwd)" ~/.cursor/plugins/local/quickstatus
```

Reload Cursor and confirm the skill and MCP server under Customize.
