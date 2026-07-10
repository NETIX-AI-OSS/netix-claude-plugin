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

## Filtering: prefer field-specific filters over `search`

Every list tool exposes typed, field-specific filters — by id, status, priority, type, category, assignee, date range, and `*_icontains` name matches. Use them; they are precise and reliable. Treat the generic `search` parameter as a last resort: it is an opaque server-side text match whose covered fields are undocumented, so it can silently miss or over-match. Whenever the tool schema has a dedicated filter for what you need (e.g. `status`, `priority`, `contract`, `fault_fault_category`, `technician_assigned`, `contract_code_icontains`, `asset_class`), use it instead of `search`. To filter by a named entity, resolve the name to an id first via the reference-data list tools rather than free-text searching by name — unless the tool has no id/`*_icontains` filter for it (e.g. vendors have no name filter, so `search` is the only option there).

## Resolving IDs to names

Records reference people and assets by numeric ID (`technician_assigned`, `engineer_assigned`, `created_by`, `performed_by`, `escalated_to`; and bare `asset_id` in analytics rows). Present names, not raw IDs:

- **Users:** collect the distinct user IDs from your results and call `user_resolve` with them (comma-separated, up to 50) to get `{id: {full_name, email, designation, …}}`. It is org-scoped and de-duplicates.
- **Assets:** most list rows already include `asset_details: {id, name, display_name}` — use that. When you only have a bare `asset_id` (e.g. from `cafm_analytics` grouped rows), resolve it via `asset_list` with `id_in` (comma-separated IDs).

Resolve once per distinct set of IDs and reuse the mapping — don't call per row.

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
4. `aggregate_by` groups results (e.g. by `status`); `time_bucket` + `time_field` produce a time series; `distinct` applies DISTINCT.
5. **Dynamic filters use Django field lookups** — this is where most of the power is. Pass `f_<field>` to include and `fe_<field>` to exclude, each with an optional `__<lookup>` suffix:
   - Ranges & dates: `f_created_on__gte: "2025-01-01"`, `f_created_on__lte: "2025-06-30"` (ISO strings for date/datetime fields).
   - Membership / text / null: `f_priority__in: "1,2"`, `f_description__icontains: "leak"`, `f_resolved_on__isnull: true`.
   - Exact match: `f_status: 3`; exclude: `fe_status: 6`. Pass several `f_`/`fe_` keys to AND them together.
   - Use the exact field names the `explain` call reports. These filters are far more precise than pulling raw rows and filtering yourself.

## Sandbox tools (per domain)

- `*_load_dataframe` then `*_execute_code`: load a dataset as a pandas DataFrame (`df`) and run Python against it. The server is stateless — **do a load and its dependent code in the flow the tools document**, and don't assume state survives across separate conversations.
- `*_generate_chart`: produce a chart from data; prefer it over describing numbers when the user asks for trends/distributions.
- These tools may be absent on some deployments (disabled server-side). If missing, fall back to list tools + your own analysis.

## Practical guidance

- Reference data (service categories, building types, asset classes) is small — fetch it whole to map ids→names before presenting results.
- Test/staging deployments may contain synthetic seed data — don't treat obviously synthetic records as real business data.
- Results can contain personal data (caller names, emails, phones). Include it only when the user's task needs it.
- If every call fails with a permission error, the `NETIX_API_KEY` env var is missing/invalid or lacks an organization — tell the user to check their key (see plugin README), don't retry.
