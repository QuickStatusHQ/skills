---
name: uptime-status-page
description: "Uptime checks, Linux server monitoring, and a public status page. Use when the user asks to monitor a health URL, see if a site is down, watch a server's CPU, memory, or disk, set up a status page, open an incident, or schedule maintenance."
---

# QuickStatus

QuickStatus probes a website, API, DNS name, or TCP host. A website check is up when the HTTP status is below 500. An API check is up on 200 unless the user set another code. A DNS check is up when the lookup succeeds. A TCP check is up when the connection succeeds, or when it fails if the user asked for that. Timeouts and other failures count as down. A server check is up while enough Linux machines with the QuickStatus.dev server agent keep reporting; it alerts on CPU, memory, disk, and processes from inside the machine.

The same workspace publishes a status page, opens incidents, and schedules maintenance. Subscriber email goes out only when the user asks.

Setup for an agent: https://quickstatus.dev/agent-setup/prompt.md. Human guide: https://quickstatus.dev/docs/mcp.

## Install

```bash
npx skills add QuickStatusHQ/skills --skill uptime-status-page -a claude-code -a codex -g -y
```

## Connect

1. If `SP_API_KEY` is already set and starts with `sp_`, keep it. Otherwise ask once. An owner or admin creates the key at https://quickstatus.dev/app/api-keys.
2. Export it in the shell. Never write the raw key into a committed file, and never print it back.
3. Wire MCP for the agent you are (idempotent). From this skill folder:

```bash
bash scripts/wire-mcp.sh cursor   # or claude, codex, all
```

If the skill lives under `~/.agents/skills/uptime-status-page`, you can run:

```bash
bash ~/.agents/skills/uptime-status-page/scripts/wire-mcp.sh codex
```

4. Call `usage` before creating a check or a page. Stop when a cap is reached. A 401 means the key is missing or revoked. Ask for a new key. Do not create checks while verifying the connection.

claude.ai, Claude Desktop, and ChatGPT expect OAuth. Do not invent a connector, a stdio package, or an OAuth client.

## After it is connected

Watch the health URLs the user names. Create checks (HTTP, API, heartbeat, MCP, DNS, TCP, or server), a status page, incidents, and maintenance from the tools.

Maintenance tools default `notify_subscribers` to false. Email subscribers only when the user asks.

Team invites, billing, creating and revoking API keys, creating or deleting alert destinations, deleting an incident, and visitor subscribers stay in the console. A private-page password belongs on Page settings, not in the chat.

## Servers (server agent)

Use a server check when the user wants to watch a Linux machine or a fleet from the inside (CPU, memory, disk, processes, "is it still running"), not a URL. Pair it with an HTTP check on the site the server runs.

1. Call `usage`. `max_server_members` is how many machines one server check can hold (Free 1, Pro 25). One server check covers a whole fleet; do not create one check per machine.
2. Call `create_check` with `kind: "server"`, a `name` such as `web-prod`, and `min_members` (how many machines must report; default 1, at most `max_server_members`). Optional: `group_grace_seconds` (default 300) so a reboot or rolling deploy does not alert. Pass no `target` and no `regions`.
3. The result has `install_command`, shown only this once. It contains the enrollment token: never write it to a file, a commit, or a config, and do not repeat it after you have used or handed it over.
4. Install the agent:
   - If you have a root shell on the server (the user said this machine is the server, or gave you SSH), ask before running the command. It installs a systemd service as root. Run it exactly as returned.
   - To alert when a process stops, add `--processes "nginx postgres"` with the names the user gives.
   - Otherwise give the user the command once and tell them to run it as root on each Linux server.
   - For autoscaling groups, Terraform, or Kubernetes, point the user to https://quickstatus.dev/docs/server-agent. Fleets read the token from SSM or Secrets Manager, not user data.
5. The installer sends the first report before it finishes. Exit 0 means it is reporting. Exit 4 means the token was rejected: the user gets a new one with New install command on the check page. Do not create a second check. Exit 7 means the check is at its plan's server limit.
6. Setup is done only when the first report is in. Call `get_check_results` with `view: "servers"` and look for a machine with `state: "active"`. It usually appears within a minute. If nothing shows after 2 minutes, run `quickstatus-agent status` on the server and check that it can reach `https://api.quickstatus.dev` over HTTPS.
7. Tell the user it is reporting, and that new checks start with three rules: CPU above 90% for 10 minutes, memory above 90% for 10 minutes, and any disk above 90%. Rules are edited in the console (Edit rules on the check page) or with the REST API, not with these tools.

For "why is prod slow?" or "which server is full?", call `get_check_results` with `view: "servers"` (latest metrics per machine and which rules are firing) or `view: "metrics"` with `hours` (the fleet over time).
