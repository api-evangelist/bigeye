---
name: Onboard a data source into Bigeye
description: Validate and create a Bigeye data source, set which schemas get metadata metrics, rebuild the catalog and wait for the workflow to finish.
api: openapi/bigeye-metadata-openapi.json
operations:
  - SourceService_ValidateSource
  - SourceService_CreateOrUpdateSource
  - SourceService_GetSource
  - SourceService_SetMetadataSchemas
  - SchemaService_SearchSchemas
  - CatalogRebuildsService_UpdateSource
  - CatalogRebuildsService_UpdateSchema
  - SourceService_GetSourceWorkflowStatus
  - WorkflowService_GetWorkflowStatus
---

# Onboard a data source into Bigeye

Use this when connecting a warehouse, lake or database to Bigeye
programmatically instead of through the UI wizard.

## Steps

1. **Validate the connection first.** `SourceService_ValidateSource` —
   `POST /api/v1/sources/validate`. This is the whole point of the flow: it
   fails cheaply before you persist bad credentials.
2. **Create it.** `SourceService_CreateOrUpdateSource` — `POST /api/v1/sources`.
   Create-or-update semantics: passing an existing source id edits rather than
   duplicating.
3. **Confirm.** `SourceService_GetSource` — `GET /api/v1/sources/{sourceId}`.
4. **Choose what gets metadata metrics.** `SourceService_SetMetadataSchemas` —
   `POST /api/v1/sources/metadata-schemas`. Discover schema ids first with
   `SchemaService_SearchSchemas` (`GET /api/v1/schemas`).
5. **Build the catalog.** `CatalogRebuildsService_UpdateSource` —
   `POST /api/v1/rebuilds/sources/{sourceId}`, or scope it down with
   `CatalogRebuildsService_UpdateSchema`
   (`POST /api/v1/rebuilds/schemas/{schemaId}`).
6. **Wait.** `SourceService_GetSourceWorkflowStatus` —
   `GET /api/v1/sources/{sourceId}/workflow/status`, or
   `WorkflowService_GetWorkflowStatus`
   (`GET /api/v1/workflows/{workflowId}/status`) when you hold a workflow id.
   The catalog is not queryable until this completes.

## Rules

- Bigeye connects with **read-only** service accounts over JDBC and extracts
  aggregate statistics, query logs and metadata — never raw rows. Do not
  configure a write-capable account.
- For in-network connections, credentials stay on the customer's infrastructure
  and are managed by the Agent CLI, not this API. See `cli/bigeye-cli.yml`.
- Poll the workflow status; do not re-issue the rebuild. There is no
  idempotency key and a second rebuild is real work on the customer's warehouse.
