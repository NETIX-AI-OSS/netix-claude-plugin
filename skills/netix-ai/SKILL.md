---
description: Query and analyze NETIX AI Platform data — assets, work orders, service requests, complaints, compliance records, contracts/vendors, and telemetry — via the netix-ai MCP server. Use whenever the user asks about their buildings, equipment, maintenance, facility records, or facility analytics.
---

# NETIX AI Platform Data

You have MCP tools (server: `netix-ai`) for the NETIX AI industrial-IoT and facility-management platform. Tenant scope is injected server-side from the user's API key — never ask for or pass an organization id; every result is already scoped to the user's organization.

## Tool domains

Tools are named `<domain>_<operation>`:

| Domain | Covers |
|---|---|
| `asset_*` | Asset registry: assets, classes, levels (hierarchy), maps. Datasets can be large (tens of thousands of assets) — always filter/limit. |
| `work_orders_*` | Service requests, reactive & PPM work orders, faults, material inventory, service/fault categories. |
| `complaints_*` | Complaints and their activities, escalations, follow-ups, linked work orders; building types, clients; plus the `cafm_analytics` aggregate tool. |
| `compliance_*` | Work permits, audit inspections, access cards/permits/parking. |
| `commercial_*` | Quotations, contracts, contract SLAs, vendors. |
| `facilities_*` | Telemetry: realtime data, historical data queries, analytics queries, alarm rule templates, prediction models. |

Most list tools accept optional filters and `limit`. Responses are shaped `{count, results, has_more}` — `count` is the TOTAL matching rows, `results` is the returned page. Lead with the count when summarizing; fetch more pages only when the user needs them.

## Work-order labor time (man-hours)

Reactive/PPM work orders and service requests can carry two labor fields:

- `manhour_secs` — elapsed labor time in **seconds** (e.g. `3600` = 1.0 hour).
- `man_hours` — a CAFM-computed value whose unit is unspecified; do **not** assume it is already in hours.

**When `manhour_secs` is set (present and non-zero) it takes priority: report labor time from it — `manhour_secs / 3600`, in hours — and treat that as authoritative.** Only fall back to `man_hours` when `manhour_secs` is absent or zero. Never present the raw `man_hours` number as an hours figure while a seconds value is available.

Example: a work order with `man_hours: 0.04` and `manhour_secs: 3600` → report **1.0 man-hour** (derived from `manhour_secs`), not 0.04.

## Analytics: `complaints_cafm_analytics`

The aggregate/reporting tool over platform records. Rules learned the hard way:

1. `model_name` is required (e.g. `"Complaint"`).
2. **First call with `explain: true`** for an unfamiliar model — it returns the model's fields, relationships, and allowed aggregations. Then build the real query.
3. If you pass `operation` (e.g. `"count"`), you MUST also pass `operation_field` (use `"id"` for row counts) — otherwise the API returns 400 "Operation field is required when operation is specified".
4. `aggregate_by` groups results (e.g. by `status`); dynamic filters use the `f_<field>` convention (e.g. `f_status: 3`).

## Sandbox tools (per domain)

- `*_load_dataframe` then `*_execute_code`: load a dataset as a pandas DataFrame (`df`) and run Python against it. The server is stateless — **do a load and its dependent code in the flow the tools document**, and don't assume state survives across separate conversations.
- `*_generate_chart`: produce a chart from data; prefer it over describing numbers when the user asks for trends/distributions.
- These tools may be absent on some deployments (disabled server-side). If missing, fall back to list tools + your own analysis.

## Practical guidance

- Reference data (service categories, building types, asset classes) is small — fetch it whole to map ids→names before presenting results.
- Test/staging deployments may contain synthetic seed data — don't treat obviously synthetic records as real business data.
- Results can contain personal data (caller names, emails, phones). Include it only when the user's task needs it.
- If every call fails with a permission error, the `NETIX_API_KEY` env var is missing/invalid or lacks an organization — tell the user to check their key (see plugin README), don't retry.
