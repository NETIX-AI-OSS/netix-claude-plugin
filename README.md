# NETIX AI Platform — Claude Code Plugin

Query your NETIX AI Platform data — assets, work orders, complaints, compliance, contracts, telemetry — from [Claude Code](https://claude.com/claude-code). Bundles the NETIX MCP server connection plus a skill that teaches Claude the platform's tools.

## Quick setup via Claude (recommended)

Paste this into any Claude Code session, with your API key filled in:

```text
Set up the NETIX AI plugin for me. My NETIX API key is: <PASTE-YOUR-KEY-HERE>

Do all of the following, verifying each step:
1. Run: claude plugin marketplace add NETIX-AI-OSS/netix-claude-plugin
2. Run: claude plugin install netix-ai@netix-ai
3. Add my API key to the "env" block of ~/.claude/settings.json as
   NETIX_API_KEY (merge with existing settings — do not remove anything,
   and validate the file is still valid JSON afterwards). This makes the
   key available to GUI/desktop sessions too, which don't read .zshrc.
4. Confirm the plugin is installed and enabled (claude plugin list).
5. Then tell me to restart Claude Code, approve the "netix-ai" MCP server
   when prompted, and give me one test question to verify it works
   (e.g. "How many asset classes do we have?").

Notes: the plugin's MCP config reads ${NETIX_API_KEY} from the environment;
the server URL defaults to NETIX production and can be overridden with a
NETIX_MCP_URL env var. Do not print my API key back in your responses.
```

Treat that message like a credential — it contains your API key and lands in your session transcript.

## Manual setup

```bash
# 1. Make your key available (add to your shell profile, or use the
#    settings.json env block as in the quick setup for GUI sessions)
export NETIX_API_KEY="<your NETIX API key>"
# Optional: point at a different NETIX environment
# export NETIX_MCP_URL="https://<your-netix-host>/mcp"

# 2. In Claude Code
/plugin marketplace add NETIX-AI-OSS/netix-claude-plugin
/plugin install netix-ai@netix-ai
```

Claude Code will ask you to approve the `netix-ai` MCP server on first use. Restart your session after setting environment variables — they are read at session startup.

Get an API key from your NETIX administrator or the NETIX user settings page. The key determines your organization: all data access is scoped server-side to your tenant.

## What you get

- **~55 MCP tools** across six domains: `asset_*`, `work_orders_*`, `complaints_*`, `compliance_*`, `commercial_*`, `facilities_*` (telemetry, analytics, alarm templates).
- **A skill** (`netix-ai`) that makes Claude effective with them: analytics query conventions, pagination semantics, reference-data mapping, and sandbox/charting tools.

Example prompts:

> How many open complaints do we have, grouped by status?

> List reactive work orders created this week for HVAC.

> Chart service requests per month for the last year.

## Notes

- No credentials are stored in this repository or the plugin — the API key comes from your environment (`NETIX_API_KEY`).
- The default server URL targets the NETIX production environment; override `NETIX_MCP_URL` for other deployments.
- Works in the Claude Code CLI, desktop app, and IDE extensions. For claude.ai web sessions (cloud VMs), use a project-scoped `.mcp.json` and the cloud environment's env-var settings instead.

## License

Licensed under the [Apache License 2.0](LICENSE).
