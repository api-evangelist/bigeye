---
name: Scan Bigeye sources for sensitive data
description: Configure data classes, create and run a Bigeye sensitive data scan job, then read the aggregate and snapshot findings.
api: openapi/bigeye-sensitivity-openapi.json
operations:
  - DataClassService_FetchDataClasses
  - DataClassService_CreateDataClass
  - ScanJobService_CreateScanJob
  - ScanJobService_SetPartitionColumns
  - ScanJobService_RunScanJob
  - ScanJobService_GetScanJobStatus
  - ScanJobService_GetAggregateFindings
  - ScanJobService_GetSnapshotFindings
  - ScanJobService_RetryScanJob
  - ScanJobService_CancelScanJob
  - ScanRunService_FetchScanRuns
---

# Scan Bigeye sources for sensitive data

Use this when asked where PII lives, to run a classification scan, or to report
on sensitive-data coverage.

## Steps

1. **Know your classes.** `DataClassService_FetchDataClasses` —
   `POST /api/v1/sds/data-class/fetch` lists the classification categories
   already configured, including the Bigeye-provided defaults. Only add one with
   `DataClassService_CreateDataClass` (`POST /api/v1/sds/data-class`) if the
   category genuinely does not exist.
2. **Create the job.** `ScanJobService_CreateScanJob` —
   `POST /api/v1/sds/scan-job`.
3. **Bound it on large tables.** `ScanJobService_SetPartitionColumns` —
   `PUT /api/v1/sds/scan-job/{id}/partition-columns` so the scan reads a
   partition instead of the whole table.
4. **Run it.** `ScanJobService_RunScanJob` —
   `POST /api/v1/sds/scan-job/{id}/run`.
5. **Poll.** `ScanJobService_GetScanJobStatus` —
   `GET /api/v1/sds/scan-job/{id}/status`. `ScanRunService_FetchScanRuns`
   (`POST /api/v1/sds/scan-run/fetch`) gives the run history.
6. **Read the findings.** `ScanJobService_GetAggregateFindings` —
   `POST /api/v1/sds/scan-job/findings/aggregate` for the rollup, and
   `ScanJobService_GetSnapshotFindings`
   (`POST /api/v1/sds/scan-job/findings/snapshot`) for a point-in-time view.
7. **Recover.** `ScanJobService_RetryScanJob`
   (`PUT /api/v1/sds/scan-job/{id}/retry`) on failure;
   `ScanJobService_CancelScanJob` (`POST /api/v1/sds/scan-job/{id}/cancel`) or
   `ScanJobService_TerminateScanJob` to stop a running scan.

## Rules

- Findings describe **where** sensitive data was detected. Do not echo sampled
  values into a chat transcript, a ticket or a log.
- Scans are expensive. Poll status rather than re-running, and never retry a
  `RunScanJob` on timeout — there is no idempotency key, so a blind retry starts
  a second scan.
