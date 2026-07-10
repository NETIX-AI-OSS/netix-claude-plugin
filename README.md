# NETIX Claude Code Plugin

Query your NETIX.AI facilities data — assets, work orders, complaints, compliance, contracts, telemetry — from [Claude Code](https://claude.com/claude-code). Bundles the NETIX MCP server connection plus a skill that teaches Claude the platform's tools.

## Install

```bash
# 1. Set your credentials (add to your shell profile)
export NETIX_API_KEY="<your NETIX API key>"
# Optional: point at a different NETIX environment
# export NETIX_MCP_URL="https://<your-netix-host>/mcp"

# 2. In Claude Code
/plugin marketplace add NETIX-AI-OSS/netix-claude-plugin
/plugin install netix@netix
```

Claude Code will ask you to approve the `netix` MCP server on first use. Restart your session after setting environment variables — they are read at session startup.

Get an API key from your NETIX administrator or the NETIX user settings page. The key determines your organization: all data access is scoped server-side to your tenant.

## What you get

- **~55 MCP tools** across six domains: `asset_*`, `work_orders_*`, `complaints_*`, `compliance_*`, `commercial_*`, `facilities_*` (telemetry, analytics, alarm templates).
- **A skill** (`netix-cafm`) that makes Claude effective with them: analytics query conventions, pagination semantics, reference-data mapping, and sandbox/charting tools.

Example prompts:

> How many open complaints do we have, grouped by status?

> List reactive work orders created this week for HVAC.

> Chart service requests per month for the last year.

## Notes

- No credentials are stored in this repository or the plugin — the API key comes from your environment (`NETIX_API_KEY`).
- The default server URL targets the NETIX production environment; override `NETIX_MCP_URL` for other deployments.

## License

Licensed under the [Apache License 2.0](LICENSE).
