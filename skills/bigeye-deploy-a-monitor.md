---
name: Deploy a Bigeye monitor on a table
description: Find a table in the Bigeye catalog, discover which metric types apply to it, create a monitor, then run and backfill it.
api: openapi/bigeye-observability-openapi.json
operations:
  - TableService_SearchTables
  - TableService_GetTables
  - TableService_TableLevelApplicableMetricTypes
  - MetricService_GetTableLevelMetricNames
  - MetricService_ValidateMetric
  - MetricService_CreateMetric
  - MetricService_RunMetric
  - MetricService_BackfillMetric
  - MetricService_GetMetricInfo
---

# Deploy a Bigeye monitor on a table

Use this when asked to "start monitoring" a table, add a freshness or volume
check, or fill a data quality gap.

## Before you start

Same auth and workspace header rules as every Bigeye call: `Authorization:
apikey <API_KEY>` plus `X-Bigeye-Workspace-Id`. Creating a metric is a write, so
the key needs an edit or manage role.

## Steps

1. **Find the table.** `TableService_SearchTables` — `GET /api/v1/tables` with
   `tableName`, `schemaName` and `warehouseIds` query parameters. When you
   already hold ids, `TableService_GetTables` (`POST /api/v1/tables/fetch`) is
   the batch form.
2. **Ask what applies.** `TableService_TableLevelApplicableMetricTypes` —
   `GET /api/v1/tables/{tableId}/applicable-metric-types` returns only the
   metric types valid for that table. `MetricService_GetTableLevelMetricNames`
   (`GET /api/v1/metrics/table-level-metric-names`) is the global list of
   table-level names such as freshness and row count. Never invent a metric
   type name — read it from one of these.
3. **Validate before you write.** `MetricService_ValidateMetric` —
   `GET /api/v1/metrics/validation` checks a metric configuration without
   creating it.
4. **Create it.** `MetricService_CreateMetric` — `POST /api/v1/metrics`. This
   operation is create-or-update: passing an existing metric id edits rather
   than duplicates, which is the closest thing Bigeye offers to a safe retry.
5. **Run it.** `MetricService_RunMetric` — `GET /api/v1/metrics/run/{metricId}`
   for one, or `MetricService_RunMetricBatchQueue`
   (`POST /api/v1/metrics/run/batch/queue`) for a large batch — that one returns
   a workflow you poll with `WorkflowService_GetWorkflowStatus`
   (`GET /api/v1/workflows/{workflowId}/status`).
6. **Give it history.** `MetricService_BackfillMetric` —
   `POST /api/v1/metrics/backfill`, then poll
   `MetricService_SearchMetricBackfill` (`GET /api/v1/metrics/backfill`) for
   status. Anomaly detection needs history before it is meaningful.
7. **Confirm.** `MetricService_GetMetricInfo` —
   `GET /api/v1/metrics/info/{metricId}` returns configuration plus latest run
   state.

## Rules

- Prefer `MetricService_CreateMetric` with an explicit id over a bare create if
  you may be re-running the flow — there is no `Idempotency-Key` header to lean
  on.
- For fleet-scale changes, Bigeye's supported path is YAML observability-as-code
  through the `bigeye` CLI rather than looping this API. See
  `cli/bigeye-cli.yml`.
