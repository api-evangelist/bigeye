# Bigeye

Bigeye is an enterprise data observability and AI trust platform that monitors data quality, detects schema changes and anomalies, classifies sensitive data, and maps column-level lineage across warehouses, lakes, BI tools and ETL pipelines.

- Website: https://www.bigeye.com/
- Documentation: https://docs.bigeye.com/
- API Reference: https://docs.bigeye.com/reference
- Status: https://status.bigeye.com/
- GitHub: https://github.com/bigeyedata

## APIs

| API | Operations | Contract |
|---|---|---|
| Bigeye Metadata API | 146 | `openapi/bigeye-metadata-openapi.json` |
| Bigeye Observability API | 86 | `openapi/bigeye-observability-openapi.json` |
| Bigeye Sensitivity API | 31 | `openapi/bigeye-sensitivity-openapi.json` |
| Bigeye MCP Gateway | 56 tools | `mcp/bigeye-mcp.yml` |

All three OpenAPI 3.0 definitions are discoverable from Bigeye's own RFC 9727 API catalog at
https://docs.bigeye.com/.well-known/api-catalog and were harvested verbatim from
`https://docs.bigeye.com/openapi/{metadata,observability,sensitivity}.json`.

The MCP gateway at `https://mcpgateway.bigeye.com/mcp` answers `tools/list` anonymously, so
`mcp/bigeye-mcp-tools-list.json` holds the provider's real tool schemas rather than inferred ones.
`mcp/bigeye-tool-crosswalk.yml` records where the MCP and REST surfaces diverge.
