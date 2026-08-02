---
name: Connect an agent to the Bigeye MCP Gateway
description: Point an MCP client at Bigeye's hosted gateway and use its 56 tools for issues, catalog search, lineage, profiling, dimensions, tags and sensitivity findings.
api: mcp/bigeye-mcp.yml
operations:
  - list_data_sources
  - list_issues
  - get_issue
  - update_issue
  - search_tables
  - get_lineage_graph
  - get_upstream_root_causes
  - get_downstream_impact
  - get_dataset_health_summary
  - get_table_sensitivity_findings
---

# Connect an agent to the Bigeye MCP Gateway

Bigeye runs a hosted MCP server at `https://mcpgateway.bigeye.com/mcp`
(public beta since July 2026). Prefer it over the REST API for conversational
work — it exposes a curated 56-tool surface, several of whose capabilities have
no published REST equivalent.

## Connect

Three request headers do all the authentication — there is no OAuth:

| Header | Value |
|---|---|
| `Authorization` | `apikey <YOUR_API_KEY>` (literal `apikey` prefix, not `Bearer`) |
| `x-bigeye-workspace-id` | your integer workspace ID |
| `x-bigeye-url` | `https://app.bigeye.com` — only on a non-default stack |

Claude Code:

```bash
claude mcp add --transport http bigeye https://mcpgateway.bigeye.com/mcp \
  --header "Authorization: apikey <YOUR_API_KEY>" \
  --header "x-bigeye-workspace-id: <YOUR_WORKSPACE_ID>"
```

Snowflake Cortex Code and GitHub Copilot CLI take the same URL and headers as an
HTTP MCP server. Verify with "Are you connected to Bigeye? List my data
sources." — that calls `list_data_sources`.

## Working flows

- **Triage:** `list_issues` → `get_issue` → `get_resolution_steps` →
  `update_issue`. `search_issues` resolves the display number shown in the UI
  to an internal id.
- **Root cause and blast radius:** `get_issue_lineage_trace` does the whole
  walk; `get_upstream_root_causes` and `get_downstream_impact` do each half.
- **Health review:** `get_dataset_health_summary`, then
  `get_table_dimension_coverage` to find which quality dimensions have no
  monitor.
- **Governance:** `list_data_classes` and `get_table_sensitivity_findings`.

## Rules

- **The MCP surface is not a wrapper over the published OpenAPI.** 21 of the 56
  tools call Bigeye endpoints that appear in no published definition — the whole
  v2 tag surface, global search, table profiling, lineage search, issue
  merge/unmerge. Conversely 241 of 263 published REST operations have no tool.
  Pick the surface by capability, not by habit. See
  `mcp/bigeye-tool-crosswalk.yml`.
- Write tools (`create_incident`, `update_issue`, `create_metric`,
  `create_dimension`, `delete_dimension`, `delete_tag`, `lineage_delete_node`)
  are real mutations with no idempotency key. Confirm with the user before
  calling one, and re-read rather than retry.
- `tools/list` is answerable anonymously, so you can enumerate the surface
  before holding credentials; tool *calls* require the key.
