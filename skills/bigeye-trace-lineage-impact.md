---
name: Trace Bigeye lineage for root cause and impact
description: Walk the Bigeye column-level lineage graph from a node to find what upstream broke and what downstream is affected.
api: openapi/bigeye-metadata-openapi.json
operations:
  - LineageV2Service_GetLineageNode
  - LineageV2Service_GetLineageGraphForNode
  - LineageV2Service_CreateCustomLineageNode
  - LineageV2Service_CreateLineageEdge
  - LineageV2Service_BulkCreateCustomLineageNodes
  - LineageV2Service_BulkCreateLineageEdge
  - LineageV2Service_DeleteLineageNode
---

# Trace Bigeye lineage for root cause and impact

Use this when an issue is open and the question is "where did this come from"
or "what breaks if I don't fix it".

## Steps

1. **Resolve the node.** `LineageV2Service_GetLineageNode` —
   `GET /api/v2/lineage/nodes/{dataNodeId}` confirms the node exists and returns
   its type and name.
2. **Pull the graph.** `LineageV2Service_GetLineageGraphForNode` —
   `GET /api/v2/lineage/nodes/{dataNodeId}/graph`. This is the single call that
   gives you both directions; read upstream for root cause and downstream for
   blast radius.
3. **Fill gaps with custom lineage.** When a hop runs through a system Bigeye
   does not parse, add it: `LineageV2Service_CreateCustomLineageNode`
   (`POST /api/v2/lineage/nodes`) then `LineageV2Service_CreateLineageEdge`
   (`POST /api/v2/lineage/edges`). Use the `*Bulk*` variants
   (`/api/v2/lineage/nodes/bulk`, `/api/v2/lineage/edges/bulk`) for more than a
   couple of hops.
4. **Clean up.** `LineageV2Service_DeleteLineageNode` —
   `DELETE /api/v2/lineage/nodes/{dataNodeId}` removes a custom node you added.

## Rules

- Only delete nodes you created. Deleting a node you did not author silently
  changes everyone's impact analysis.
- Several lineage capabilities the first-party MCP server uses — node search
  (`POST /api/v2/lineage/search`), a node's issues
  (`GET /api/v2/lineage/nodes/{id}/issues`), report upstream issues — are **not
  in any published OpenAPI definition**. If you need them, go through the MCP
  gateway rather than the REST API. See `mcp/bigeye-tool-crosswalk.yml`.
